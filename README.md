# utl_rename_variables_using_coordinated_lists_renamel_macro
Rename tables using coordinated lists of variable names.  
    Rename tables using coordinated lists of variable names

      Two Solutions

         1. DOSUBL
         2. Ian Whitlock utl_renamel macro  (macro on end and in github)


    github (macro in project and at the end of this doc)
    https://goo.gl/MY6YTw
    https://github.com/rogerjdeangelis/utl_rename_variables_using_coordinated_lists_renamel_macro

    Original topic:  Rename variable names using macro or array

    https://goo.gl/m1kvU5
    https://communities.sas.com/t5/General-SAS-Programming/Rename-variable-names-using-macro-or-array/m-p/435744


    INPUT
    =====

     ALGORITHM

      Rename variables in work.oldnames using variable names in sashelp.class
      Create new dataset work.newNames renaming

      WORK.OLDNAMES

       Variables in Creation Order

      #    Variable  |  RULES
                     |
      1    NAM       |  ** rename to NAME
      2    GENDER    |  ** rename to SEX
      3    YRS       |  ** rename to AGE
      4    HGT       |  ** rename to HEIGHT
      5    WGT       |  ** rename to WEIGHT


      SASHELP.CLASS (has the names I want)

        Variables in Creation Order

       #    Variable

       1    NAME     NAM
       2    SEX      GENDER
       3    AGE      YRS
       4    HEIGHT   HGT
       5    WEIGHT   WGT


    PROCESS
    =======

     1. DOSUBL


       data newNames;

        if _n_=0 then do;
          %let rc=%sysfunc(dosubl('
              proc transpose data=sashelp.class(obs=1) out=xpoCls;var _all_;run;quit;
              proc transpose data=oldNames(obs=1)      out=xpoFix;var _all_;run;quit;
              data _null_;
                 length cmdRen $16000;
                 retain cmdRen;
                 merge xpoCls xpoFix(rename=_name_=fix);
                 cmdRen=catx(" ",cmdRen,fix,"=",_name_);
                 call symputx("cmdRen",cmdRen);
              run;quit;
              '))
         ;
         end;
         set oldNames(rename=(&cmdRen));

       run;quit;


     2. IAN WHITLOCK RENAMEL MACRO

        data newNames;

          set oldNames (rename=(
             %utl_renamel(nam gender yrs hgt wgt ,
                         name sex age height weight) ) );
        run;quit;


    OUTPUT  newNames table created by renaming varibles in oldname table
    ======


    p to 40 obs WORK.NEWNAMES total obs=19

    bs    NAME       SEX    AGE    HEIGHT    WEIGHT

     1    Alfred      M      14     69.0      112.5
     2    Alice       F      13     56.5       84.0
     3    Barbara     F      13     65.3       98.0
     4    Carol       F      14     62.8      102.5
     ....

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

      * use sashelp.class for newnames;

      data oldNames;
         set sashelp.class(rename=(
           NAME   = NAM
           SEX    = GENDER
           AGE    = YRS
           HEIGHT = HGT
           WEIGHT = WGT));
      run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __  ___
    / __|/ _ \| | | | | __| |/ _ \| '_ \/ __|
    \__ \ (_) | | |_| | |_| | (_) | | | \__ \
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|___/

    ;

    see above

    *
     _ __ ___   __ _  ___ _ __ ___
    | '_ ` _ \ / _` |/ __| '__/ _ \
    | | | | | | (_| | (__| | | (_) |
    |_| |_| |_|\__,_|\___|_|  \___/

    ;

    /* Renameing long corrdinated lists of variables %renamel(old = a b c , new = x y z)

    /* From: Ian Whitlock <whitloi1@WESTATPO.WESTAT.COM> */


    %macro utl_renamel ( old , new ) ;
        /* Take two cordinated lists &old and &new and  */
        /* return another list of corresponding pairs   */
        /* separated by equal sign for use in a rename  */
        /* statement or data set option.                */
        /*                                              */
        /*  usage:                                      */
        /*    rename = (%renamel(old=A B C, new=X Y Z)) */
        /*    rename %renamel(old=A B C, new=X Y Z);    */
        /*                                              */
        /* Ref: Ian Whitlock <whitloi1@westat.com>      */

        %local i u v warn ;
        %let warn = Warning: RENAMEL old and new lists ;
        %let i = 1 ;
        %let u = %scan ( &old , &i ) ;
        %let v = %scan ( &new , &i ) ;
        %do %while ( %quote(&u)^=%str() and %quote(&v)^=%str() ) ;
            &u = &v
            %let i = %eval ( &i + 1 ) ;
            %let u = %scan ( &old , &i ) ;
            %let v = %scan ( &new , &i ) ;
        %end ;

        %if (null&u ^= null&v) %then
            %put &warn do not have same number of elements. ;

    %mend  utl_renamel ;


