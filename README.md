# utl-remove-all-duplicates-after-concatenating-two-tables-each-with-350-variables-and-500-thousand-ob
Remove all duplicates after concatenating two tables each with 350 variables and 500 thousand obs
    Remove all duplicates after concatenating two tables each with 350 variables and 500 thousand obs

    * My solution took about 15 seconds, SAS has a threaded sort.;

    GitHub
    https://tinyurl.com/beemtc5u
    https://github.com/rogerjdeangelis/utl-remove-all-duplicates-after-concatenating-two-tables-each-with-350-variables-and-500-thousand-ob

    SAS-L
    https://listserv.uga.edu/cgi-bin/wa?A2=SAS-L;73e3a5b6.2103a

    Output only unique records after concatenating two tables .
    In other words drop all 'duplicate groupings'.
    Keep only the onsies.
    I was going to use systask and run parallel sorts, but even
    on my very slow Dell E6420 laptop, it took less than 30 seconds.

    data have1 have2;
      retain nob fro;
      array n[175];
      array ll[175] $ 4;
      do j=1 to 1000000;
         do I = 1 to dim(n);
           n[I] = int(900*ranuni(17)) + 1000;
           ll[I] = cats("C", I, _N_);
         end;
         select;
          when ( mod(j,3)=0) do;nob=j;fro="HAVE1";output have1;fro="HAVE2";output have2;;end;
          when ( mod(j,7)=0) do;nob=j;fro="HAVE1";output have1;end;
          when ( mod(j,5)=0) do;nob=j;fro="HAVE2";output have2;end;
          otherwise ;
         end;
      end;
      drop I j;
    run;
    ;;;;
    run;quit;
    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;
    * I HAVE COMBINED AND SORTED THE TABLES;
    * THIS IS JUST TO EXPLAIN THE PROCESS;
    * I DROP THE FRO AND NOB VARIABLES IN THE SOLUTION;

    data concat/view=concat;
       set
           have1 have2;
    run;quit;

    proc sort data=concat out=document;
    by _all_;
    run;quit;

    /*
    WORK.DOCUMENT total obs=876,190

     FRO  NOB  N1   N2   N3  ... N173 N174 N175    LL1  LL2  LL3  ... LL173 LL174   LL175

    HAVE1   3 1769 1700 1894 ... 1022 1343 1887    C11  C21  C31  ...  C173  C174    C175 * Drop this dup group;
    HAVE2   3 1769 1700 1894 ... 1022 1343 1887    C11  C21  C31  ...  C173  C174    C175

    HAVE2   5 1320 1088 1230 ... 1734 1499 1487    C11  C21  C31  ...  C173  C174    C175 * keep;

    HAVE1   6 1082 1375 1756 ... 1105 1495 1812    C11  C21  C31  ...  C173  C174    C175 * Drop this dup group;
    HAVE2   6 1082 1375 1756 ... 1105 1495 1812    C11  C21  C31  ...  C173  C174    C175

    HAVE1   7 1556 1157 1437 ... 1264 1755 1118    C11  C21  C31  ...  C173  C174    C175  * keep;

    HAVE1   9 1525 1690 1130 ... 1772 1073 1534    C11  C21  C31  ...  C173  C174    C175  * Drop this dup group;
    HAVE2   9 1525 1690 1130 ... 1772 1073 1534    C11  C21  C31  ...  C173  C174    C175

    HAVE2  10 1724 1265 1688 ... 1274 1348 1437    C11  C21  C31  ...  C173  C174    C175  *keep;

    HAVE1  12 1305 1383 1832 ... 1286 1523 1035    C11  C21  C31  ...  C173  C174    C175 * Drop this dup group;
    HAVE2  12 1305 1383 1832 ... 1286 1523 1035    C11  C21  C31  ...  C173  C174    C175
    */

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    ;

    * just in case you rerun;
    proc datasets lib=work nolist mt=data mt=view;
      delete concat unique_rows;
    run;quit;

    data concat/view=concat;
       set
           have1 have2;
    run;quit;

    * NOTE I DROP THE KEY VARIABLES;

    proc sort data=concat(drop=nob fro) out=_null_ uniqueout=unique_rows nouniquekey;
    by  _all_;
    run;quit;

    *_
    | | ___   __ _
    | |/ _ \ / _` |
    | | (_) | (_| |
    |_|\___/ \__, |
             |___/
    ;

    /*
    654   data concat/view=concat;
    655      set
    656          have1 have2;
    657   run;

    NOTE: DATA STEP view saved on file WORK.CONCAT.
    NOTE: A stored DATA STEP view cannot run under a different operating system.
    NOTE: DATA statement used (Total process time):
          real time           0.03 seconds
          cpu time            0.01 seconds


    657 !     quit;
    658   proc sort data=concat(drop=nob fro) out=_null_ uniqueout=unique_rows nouniquekey;
    659   by  _all_;
    660   run;

    NOTE: There were 876190 observations read from the data set WORK.CONCAT.
    NOTE: View WORK.CONCAT.VIEW used (Total process time):
          real time           4.30 seconds
          cpu time            7.95 seconds

    NOTE: There were 428571 observations read from the data set WORK.HAVE1.
    NOTE: There were 447619 observations read from the data set WORK.HAVE2.
    NOTE: 209524 observations with unique key values were deleted.
    NOTE: The data set WORK.UNIQUE_ROWS has 209524 observations and 350 variables.
    NOTE: PROCEDURE SORT used (Total process time):
          real time           8.02 seconds
          cpu time            14.82 seconds
    */

    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;

    /*
    WORK.UNIQUE_ROWS total obs=209,524

       Obs  N1   N2   N3         N172 N173 N174    LL1 LL2 LL3    LL173   LL174   LL175
                          ...
         1 1000 1000 1756 ...    1230 1261 1880    C11 C21 C31    C173    C174    C175
         2 1000 1002 1508 ...    1747 1723 1156    C11 C21 C31    C173    C174    C175
         3 1000 1005 1766 ...    1683 1483 1867    C11 C21 C31    C173    C174    C175
         4 1000 1007 1135 ...    1045 1098 1398    C11 C21 C31    C173    C174    C175
         5 1000 1007 1797 ...    1748 1696 1768    C11 C21 C31    C173    C174    C175
         6 1000 1008 1580 ...    1647 1518 1019    C11 C21 C31    C173    C174    C175
         7 1000 1010 1809 ...    1490 1819 1002    C11 C21 C31    C173    C174    C175
         8 1000 1011 1735 ...    1574 1813 1470    C11 C21 C31    C173    C174    C175
         9 1000 1014 1000 ...    1146 1635 1386    C11 C21 C31    C173    C174    C175
        10 1000 1017 1585 ...    1727 1550 1348    C11 C21 C31    C173    C174    C175

    * as a weak check lets see if this  record occurs only once;

    HAVE2   5  1320 1088 1230 ... 1734 1499 1487

    data once;
      set UNIQUE_ROWS;
      if N1   = 1320 and
         N2   = 1088 and
         N3   = 1230 and
         N4   = 1734 and
         N5   = 1499 and
         N6   = 1487 then output;
    run;quit;

    NOTE: There were 209524 observations read from the data set WORK.UNIQUE_ROWS.

    NOTE: The data set WORK.ONCE has 1 observations and 350 variables.

    NOTE: DATA statement used (Total process time):
          real time           0.27 seconds
          user cpu time       0.03 seconds
    */
    ;;;;
    run;quit;
