# utl-first-dot-functionality-in-r-sas-and-wps
First dot functionality in r sas and wps 
    %let pgm=utl-first-dot-functionality-in-r-sas-and-wps;

    First dot functionality in r sas and wps

       Four solutions
           1. SAS
           2. SAS drop down to r ( I don't have IML)
           3. WPS
           4. WPS proc r (part of base WPS)


    github
    https://tinyurl.com/yv764mut
    https://github.com/rogerjdeangelis/utl-first-dot-functionality-in-r-sas-and-wps

    StackOverflow R
    https://tinyurl.com/3zkh8snh
    https://stackoverflow.com/questions/73789418/calculate-diffs-per-group-with-default-from-other-column

    macros
    https://tinyurl.com/y9nfugth
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories

    /*          _
     _ __ _   _| | ___  ___
    | `__| | | | |/ _ \/ __|
    | |  | |_| | |  __/\__ \
    |_|   \__,_|_|\___||___/

    */                          | RULES         Diff Values
                                |
    Obs    GRP     CAT     VAL  |  GRP     CAT     VAL
                                |
     1      A     one        2  |   A     one        2   2           leave alone
     2      A     two       10  |   A     two        8   10-2  = 8
     3      A     three     20  |   A     three     10   20-10 = 10

     4      B     one        7  |   B     one        7   7           leave alone
     5      B     two       11  |   B     two        4   11-7  = 4
     6      B     three     19  |   B     three      8   19-11 = 8


    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options valivarname=upcase; * R is case sensitive this siplifies the R code;
    data sd1.have;
     input grp $ cat $ val;
    cards4;
    A one 2
    A two 10
    A three 20
    B one 7
    B two 11
    B three 19
    ;;;;
    run;quit;

    Up to 40 obs WORK.HAVE total obs=6 20SEP2022:10:36:27

    Obs    GRP     CAT     VAL

     1      A     one        2
     2      A     two       10
     3      A     three     20
     4      B     one        7
     5      B     two       11
     6      B     three     19

    /*
     _
    / |    ___  __ _ ___
    | |   / __|/ _` / __|
    | |_  \__ \ (_| \__ \
    |_(_) |___/\__,_|___/

    */


    data want;
      set sd1.have;
      by grp;
      dif = dif(val);
      if first.grp then dif = val;
      output;
    run;quit;

    Up to 40 obs WORK.WANT total obs=6 20SEP2022:10:39:37

    Obs    GRP     CAT     VAL    DIF

     1      A     one        2      2
     2      A     two       10      8
     3      A     three     20     10
     4      B     one        7      7
     5      B     two       11      4
     6      B     three     19      8

    /*___
    |___ \     ___  __ _ ___   _ __
      __) |   / __|/ _` / __| | `__|
     / __/ _  \__ \ (_| \__ \ | |
    |_____(_) |___/\__,_|___/ |_|

    */

    * note the as.data.frame. I wish R would not turn dataframes to less useful objects.;
    * the easiest way to work within 'dataframes' is to use sqldf or sqlite. Shunned by R and Python?;

    %utlfkil(d:/xpt/want.xpt);

    %utl_submit_r64('
    library(haven);
    library(SASxport);
    library(dplyr);
    have <- read_sas("d:/sd1/have.sas7bdat");
    want<-have %>%
      group_by(GRP) %>%
      mutate(across(VAL, ~ c(first(.x), diff(.x))));
    want<-as.data.frame(want);
    write.xport(want,file="d:/xpt/want.xpt");
    ');

    libname xpt xport "d:/xpt/want.xpt";

    proc print data=xpt.want;
    run;quit;

    Obs    GRP     CAT     VAL

      1     A     one        2
      2     A     two        8
      3     A     three     10
      4     B     one        7
      5     B     two        4
      6     B     three      8

    /*____
    |___ /   __      ___ __  ___
      |_ \   \ \ /\ / / `_ \/ __|
     ___) |   \ V  V /| |_) \__ \
    |____(_)   \_/\_/ | .__/|___/
                      |_|
    */

    %utl_submit_wps64('
    libname sd1 "d:/sd1";
    data want;
      set sd1.have;
      by grp;
      dif = dif(val);
      if first.grp then dif = val;
      output;
    run;quit;
    proc print data=want;
    run;quit;
    ');

    The WPS System

    Obs    GRP     CAT     VAL    DIF

     1      A     one        2      2
     2      A     two       10      8
     3      A     three     20     10
     4      B     one        7      7
     5      B     two       11      4
     6      B     three     19      8

    /*  _
    | || |    __      ___ __  ___   _ __  _ __ ___   ___   _ __
    | || |_   \ \ /\ / / `_ \/ __| | `_ \| `__/ _ \ / __| | `__|
    |__   _|   \ V  V /| |_) \__ \ | |_) | | | (_) | (__  | |
       |_|(_)   \_/\_/ | .__/|___/ | .__/|_|  \___/ \___| |_|
                       |_|         |_|
    */

    %utlfkil(d:/xpt/want.xpt);

    %utl_submit_wps64('
    proc r;
    submit;
    library(haven);
    library(SASxport);
    library(dplyr);
    have <- read_sas("d:/sd1/have.sas7bdat");
    want<-have %>%
      group_by(GRP) %>%
      mutate(across(VAL, ~ c(first(.x), diff(.x))));
    want<-as.data.frame(want);
    want;
    write.xport(want,file="d:/xpt/want.xpt");
    endsubmit;
    ');

    libname xpt xport "d:/xpt/want.xpt";

    proc print data=xpt.want;
    run;quit;

    The WPS System

      GRP   CAT VAL
    1   A   one   2
    2   A   two   8
    3   A three  10
    4   B   one   7
    5   B   two   4
    6   B   three 8

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
