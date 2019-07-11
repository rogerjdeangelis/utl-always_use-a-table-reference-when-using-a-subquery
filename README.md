# utl-always_use-a-table-reference-when-using-a-subquery
Always use table references within subqueries?

    StackOverflow SAS: Always use table references within subqueries?

    If the variable of the same name in the transaction subquery does not exist then SAS will
    return all rows in the master table.

    Problem
       Select name = "Roger' from the master table when there is no column 'name' in the subquery.

           Two attempted solutions

             a. Incorrect Solution (returns all observations in the master - no warning or error)
             b. Correct Solution fails with an error message)


    Possible explanation:

    If the column referenced in the inner subquery does not exist use the names and values from the outer table.
    An inner join may be faster or hash might be faster and less cpu?


    SAS-L which references StackOverflow
    https://listserv.uga.edu/cgi-bin/wa?A2=SAS-L;11810443.1907b

    Question and answered by
    Joe Matise <snoopy369@GMAIL.COM>  et all

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;
    data master;
      set sashelp.class(keep=name age sex where=(name=:"J"));
    run;quit;

    data transaction;
      set sashelp.classfit(keep=name age sex predict
           where=(name in ('Jane','John')));
      drop name;
    run;quit;


    WORK.MASTER total obs=7

    Obs    NAME       SEX    AGE

     1     James       M      12
     2     Jane        F      12
     3     Janet       F      15
     4     Jeffrey     M      13
     5     John        M      12
     6     Joyce       F      11
     7     Judy        F      14


    Up to 40 obs WORK.TRANSACTION total obs=2

    Obs    SEX    AGE    PREDICT

     1      M      12    87.0159
     2      F      12    90.1351

    *            _               _
      ___  _   _| |_ _ __  _   _| |_ ___
     / _ \| | | | __| '_ \| | | | __/ __|
    | (_) | |_| | |_| |_) | |_| | |_\__ \
     \___/ \__,_|\__| .__/ \__,_|\__|___/
                    |_|
               _                                    _
      __ _    (_)_ __   ___ ___  _ __ _ __ ___  ___| |_
     / _` |   | | '_ \ / __/ _ \| '__| '__/ _ \/ __| __|
    | (_| |_  | | | | | (_| (_) | |  | | |  __/ (__| |_
     \__,_(_) |_|_| |_|\___\___/|_|  |_|  \___|\___|\__|

    ;

    proc sql;
      create
          table wrong as
      select
          *
      from
          master
      where
          name in  /* column name is not in transaction table */
            (
              select
                  name
              from
                  transaction
             where

            )
    ;quit;


    Up to 40 obs WORK.WRONG total obs=7

    Obs    NAME       SEX    AGE

     1     James       M      12
     2     Jane        F      12
     3     Janet       F      15
     4     Jeffrey     M      13
     5     John        M      12
     6     Joyce       F      11
     7     Judy        F      14

    Ho warning or error;

    NOTE: Table WORK.WRONG created, with 7 rows and 3 columns.

    59  !  quit;
    NOTE: PROCEDURE SQL used (Total process time):
          real time           0.01 seconds
          cpu time            0.00 seconds


     *_                                     _
    | |__      ___ ___  _ __ _ __ ___  ___| |_
    | '_ \    / __/ _ \| '__| '__/ _ \/ __| __|
    | |_) |  | (_| (_) | |  | | |  __/ (__| |_
    |_.__(_)  \___\___/|_|  |_|  \___|\___|\__|

    ;

    proc sql;
      create
          table wrong as
      select
          *
      from
          master
      where
          name in  /* column name is not in transaction table */
            (
              select
                  t.name
              from
                  transaction as t
            )
    ;quit;

     ERROR: Column name could not be found in the table/view identified with the correlation name T.
     ERROR: Unresolved reference to table/correlation name t.
     76  !  quit;
     NOTE: The SAS System stopped processing this step because of errors.
     NOTE: PROCEDURE SQL used (Total process time):
           real time           0.00 seconds
           cpu time            0.00 seconds

