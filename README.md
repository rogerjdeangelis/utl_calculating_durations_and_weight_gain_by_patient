# utl_calculating_durations_and_weight_gain_by_patient
Calculating durations and weight gain by patient. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    Calculating durations and weight gain by patient

    github
    https://tinyurl.com/yb698fx2
    https://github.com/rogerjdeangelis/utl_calculating_durations_and_weight_gain_by_patient

      All methods gave exactly the same results in WPS and SAS

      Four solutions

         1. Paul Dorfhman SQL
         2. Paul Dorfman datastep
         3. Mark Keintz  sum(0,dif(var2))  clever?
         4. Roger DeAngelis


    INPUT
    =====

    WORK.HAVE total obs=7  (I format date as days since 1/JAN1960)

                            |   RULES
                            |
      ID    VAR1     VAR2   | ID    CNT         DIFFVAR     DURATION
                            |
       1     11     20821   |       # records   13-11=2    20834-20821=3
       1     12     20828   |
       1     13     20834   | 1     3              2          13
                            |
       2     12     20851   |
       2     14     20863   |
       2     15     20878   |
       2     17     20893   |


    EXAMPLE OUTPUT

      CNT    ID    DIFFVAR    DURATION

       3      1       2          13
       4      2       5          42


    PROCESS
    =======


    1. Paul Dorfhman SQL

       proc sql ;
        create table want as
        select id
             , max(var1) - min(var1) as diffvar
             , max(var2) - min(var2) as duration
             , count (*)             as count
        from   have
        group  id
        ;
       quit ;


    2. Paul Dorfman datastep

       If the data is sorted (as in your sample), it may be faster
       run-time wise to go along the path of a variant of what Dan has offered:

       data want (keep = id diffvar duration count) ;
         do count = 1 by 1 until (last.id) ;
           set have ;
           by id ;
           if count = 1 then do ;
             first1 = var1 ;
             first2 = var2 ;
           end ;
         end ;
         diffvar  = var1 - first1 ;
         duration = var2 - first2 ;
       run ;


    3. Mark Keintz  sum(0,dif(var2))

       data want (drop=var:);
         set have;
         by id;
         if first.id or last.id then do;
           diffvar=sum(0,dif(var1));
           duration=sum(0,dif(var2));
         end;
         if last.id;
         cnt=min(_n_,dif(_n_));
       run;

     4. Roger DeAngelis

        data want;
         retain var1_beg var2_beg cnt .0;
          set have;
          by id;
          cnt=cnt+1;
          if first.id then do;
              var1_beg=var1;
              var2_beg=var2;
          end;
          if last.id then do;
             diffvar=var1-var1_beg;
             duration =var2 - var2_beg;
             keep id diffvar duration cnt ;
             output;
             cnt=0;
          end;
        run;quit;

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;
    data have;
    input id var1 var2 mmddyy10.;
    cards4;
    1 11 1/2/2017
    1 12 1/9/2017
    1 13 1/15/2017
    2 12 2/1/2017
    2 14 2/13/2017
    2 15 2/28/2017
    2 17 3/15/2017
    ;;;;
    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __  ___
    / __|/ _ \| | | | | __| |/ _ \| '_ \/ __|
    \__ \ (_) | | |_| | |_| | (_) | | | \__ \
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|___/

    ;

    %utl_submit_wps64('
    libname wrk sas7bdat "%sysfunc(pathname(work))";

    data wantrd;
     retain var1_beg var2_beg cnt .0;
      set wrk.have;
      by id;
      cnt=cnt+1;
      if first.id then do;
          var1_beg=var1;
          var2_beg=var2;
      end;
      if last.id then do;
         diffvar=var1-var1_beg;
         duration =var2 - var2_beg;
         keep id diffvar duration cnt ;
         output;
         cnt=0;
      end;
    run;quit;
    proc print;
    run;quit;

    data wantpd1 (keep = id diffvar duration count) ;
      do count = 1 by 1 until (last.id) ;
        set wrk.have ;
        by id ;
        if count = 1 then do ;
          first1 = var1 ;
          first2 = var2 ;
        end ;
      end ;
      diffvar  = var1 - first1 ;
      duration = var2 - first2 ;
    run ;
    proc print;
    run;quit;

    proc sql ;
      select id
           , max(var1) - min(var1) as diffvar
           , max(var2) - min(var2) as duration
           , count (*)             as count
      from   wrk.have
      group  id
      ;
    quit ;

    data wantmk (drop=var:);
      set wrk.have;
      by id;
      if first.id or last.id then do;
        diffvar=sum(0,dif(var1));
        duration=sum(0,dif(var2));
      end;
      if last.id;
      cnt=min(_n_,dif(_n_));
    run;
    proc print;
    run;quit;
    ');
