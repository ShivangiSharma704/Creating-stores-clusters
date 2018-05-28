# Creating-stores-clusters
Aim is to create store clusters based on mix of sales by categories and avg slaes per sqr foot of space
Proc import datafile ="/home/sharmashivangi7040/clustering_data-class14.csv"
DBMS=csv
out=Clustering;
run;

Data Cluster;
infile"/home/sharmashivangi7040/clustering_data-class14.csv" dsd dlm="," missover firstobs=2;
input Store_Num	Cat1	Cat2	Cat3	Cat4	Size	Sale	State$;
run;

*checking the missing values and the outliers;
proc contents data=Cluster;
run;

proc freq data=cluster;
run;

proc means N Nmiss median mean std P1 P99 data=cluster;
run;

Proc univariate data=cluster nextrobs=20;
run;

Data cluster1;
set cluster;
if cat3=212 then cat3=180;
if sale=838 then sale=800;
run;

proc means std mean median data=cluster1;
var cat3 sale;
run;


***Additional variable*;

Data Add;
set Cluster;
avg_sale=(Sale/size);
Pcat1=Cat1/sum(cat1,cat2,cat3,cat4);
Pcat2=cat2/sum(cat1,cat2,cat3,cat4);
Pcat3=Cat3/sum(cat1,cat2,cat3,cat4);
Pcat4=Cat4/sum(cat1,cat2,cat3,cat4);
run;

Proc means data=Add;
var Avg_sale Pcat1-Pcat4;
run;

*Scaling*;

Proc standard data =Add mean=0 std=1
out=scaled;
var avg_sale Pcat1-Pcat4;
run;

Data weighted;
set scaled;
avg_sales3=avg_sale*3;
run;

Proc fastclus data=weighted out=cluster6 maxclusters=6;
var avg_sales3 Pcat1-pcat4;
run;

*adding the cluster value to the original dataset;

data cluster6;
set cluster6;
keep store_num cluster;
run;
proc sort data=cluster6;
by store_num;
run;
proc sort data=add;
by store_num;
run;
data cluster_merge;
merge cluster6(in=a) add(in=b);
by store_num;
if (a and b);
run;

proc sort data=cluster_merge;
by cluster;
run;
proc means data=cluster_merge;
var avg_sale Pcat1-pcat4;
by cluster;
run;

*Area distribution of the cluster;

proc freq data=cluster_merge;
table cluster*state/norow nocol;
run;

*Avg area of cluster;
proc means data=cluster_merge;
var size;
by cluster;
run;
