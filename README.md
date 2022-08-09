/*Concomitatnt medications-------CM
other than study drugs
Topic Variable-------CMTRT
Struture: On record/Medication/constant-dosing interval/subject

Studyid
domain
usubjid
cmseq
CMCAT--------Category of the medication (prior Concomitant medication or during Concomitant medication)

CMSTDTC<RFSTDTC Then CMCAT="PRIOR";
CMSTDTC>=RFSTDTC Then CMCAT="CM";

CMSCAT-----------SUB CATEGORY

CMTRT-------Verabtium Term of the medication
CMMODIFY------Modified text
CMDECOD-------Standard dictionary derived term(WHO)
CMDOSE-------Dose
CMDOSFRQ-------Frequency
CMDOSU---------Units of the medication
CMROUTE--------Route of the administration
CMINDC----------Indication
CMDOSTOT--------Total Daily dose
CMDOSTXT--------Text Description

CMPRESP------Prespecified medications which specified on CRF or the protocol
CMOCCUR------Occurance (y/n) or Null
CMSTAT------Status(Not done)
CMREASND------ Reason for not done

CMSTDTC--------start date of the medication
CMENDTC--------End date and time of the medication
CMSTDY--------Start study day collection
CMENDY--------ENd study day collection
CMSTRF-------Start reference period of the medication
CMENRF-------End reference period of the medication
***************************************************************************************
/*Concomitatnt#2*/
/*Create Format for Freq*/

options validvarname=upcase missing;
Proc format;
invalue ft "3 per week"=0.42
"4 tabs weekly"=0.57
"4 tabs weekly"=0.57
"6 X WK"=0.85
"6 X WK"=0.85
"6 times per day"=6
"6xwk"=0.85
"75cc/H"=.
"8 times per day"=8
"BID"=2
"BID, PRN"=.
"BID,PRN"=.
"Cont. Inf."=
"Cont. drip"=.
"Continuous"=.
"Every 2 weeks"=0.07
"HS"=0.5
"ND"=.
"OD"=1
"Once"=1
"Once per Hour"=24
"Once per Month"=0.03
"Once per week"=0.14
"other: 2doses"=2
"other: 2 x wk"=0.14
"other:2xWK"=0.14
"Other:72 hrs"=0.33
"Other:72 hrs."=0.33
"other:AC & HS"=0.5
"Other:Q 3 days"=0.33
"Other:Q 10 days"=0.1
"Other: Q21day"=0.51
"Other:Q2h"=12
"Other:Q2wk"=0.07
"other:Q6WK"=0.02
"Other:Q72h"=0.33
"Other:QAM"=1
"Other:QOW"=0.14
"Other: QPM"=1
"Other: Qhd"=.
"Other: Qhr"=24
"Other: Qow"=0.07
"Other: Qwk"=0.14
"Other: x4"=.
"Other: q72h"=0.33
"Other: Qhd"=.
"Other:2am 1pm"=2
"Other:C12h and"
"Other:Every2wks"=0.07
"Other:Once/Cycl"=.
"Other:Oncew/che"=.
"other:Q 3 days"=0.33
"other:Q 72 Hr"=0.33
"Other: Q2 weeks"=0.07
"Other:Q2 weeks"=0.07
"Other:Q3 weeks"=0.51
"Other:Q3-4 weeks"=.
"Other:Q3months"=0.01
"Other:Q3weeks"=0.51
"Other:Q3wk"=0.51
"Other:Q72H"=0.33
"Other:Q72h"=0.33
"Other:QOW"=0.07
"Other:Qhd"=.
"Other:Qowk"=0.07
"Other: q 3 days"=0.33
"Other:q2 weeks"=0.07
"Other:q2h"=12
"Other:Q72h"=0.33
"Other:xlw/chemo"=.
"PRN"=.
"PRN qhs"=.
"Q other wk"=0.07
"Q4H"=6
"Q4H, PRN"=.
"Q4H, PRN"=.
"Q4H, PRN"=.
"Q6H"=4
"Q6h,PRN"=.
"Q72H"=0.33
"QD"=1
"QD,PRN"=.
"QD,PRN"=.
"QHS"=1
"QID"=4
"QID,PRN"=.
"QID,PRN"=.
"QOD"=0.5
"QOW"=0.07
"QW"=0.14
"QWK"=0.14
"TID"=3
"TID PRN"=.
"Twice per week"=0.28
"UNK"=.
"UNKNOWN"=.
"UNK"=.
"Weekly"=0.14
"am"=1
"pm"=1
"qhs"=1
"unk"=.
"xl"=.
"qpm"=1;
run;

/*Macro for sorting*/
%Macro sort (ds=, var=);
proc sort data=&ds; by &var; run;
%mend;


data cm1 (keep=studyid domain usubjid cmtrt cmmodify cmcat cmscat cmindc cmclascd cmdose cmdoseu
cmdosfrq cmroute cmclas cmclascd cmdose cmdostot cmdoseu cmdosfrq cmroute cmstdtc cmendtc
set raw.nsmed (drop=studyid);
studyid=study;
domain='CM';
usubjid=catx('-', study, stdysite, patient);
cmtrt=medx;
cmmodify=medptx;
cmdecod=medptx; or /*PT in who dictionary*/
cmcat=upcase(lblstyp);
cmscat=upcase(tmttyp);
cmindc=tmtrsnx;
cmclas=atc4x;
cmclascd=atc4c;
cmdose=dose;
cmdoseu=doseu;
cmdosfrq=freq;
if freq ne ' ' then
frq=input(freq,ft.);
comdostot=cmdose;
cmroute=route;
if startdt ne . then do;
if startdtp='DY' then cmstdtc=put(datepart(startdt),Is8601da.);
else if startdtp="MO" then cmstdtc=put(datepart(startdt), Is8601da.);
else if startdp="YR" then cmstdtc=put(datepart(startdt), Is8601da.);
end;
if stopdt ne . then do;
if stopdtp='DY' then cmendtc=put(datepart(stopdt),Is8601da.);
else if stopdtp="MO" then cmendtc=put(datepart(stopdt), Is8601da.);
else if stopdp="YR" then cmendtc=put(datepart(stopdt), Is8601da.);
end;
run;

%sort (ds=cm1, var=usubjid);
proc sort data=sdt.sdtm_dm out=dm(keep=rfstdtc rfendtc usubjid)l
by usubjid;
run;

/*deriving studyday and epoch*/
data cm_dm(drop=rfstdtc rfendtc rfendt rfsdt cmstdt cmendt);
merge cm1 (in=a) dm;
by usubjid;
if a;
if cmstdtc ne ' ' then cmstdt=input(cmstdtc, is8601da.);
if rfstdtc ne ' ' then rfstdt=input(rfstdtc, is8601da.);
if rfendtc ne ' ' then rfendt=input(rfendtc, is8601da.);
if cmendtc ne ' ' then cmendt=input(cmendtc, is8601da.);

if nmiss(cmstdt,rfstdt)=0 then do;
if cmstdt<rfstdt then cmstdy=cmstdt-rfstdt;
else if cmstdt ge rfstdt then cmstdy=(cmstdt-rfstdt)+1;
end;
if nmiss(cmendt,rfstdt)=0 then do;
if cmendt<rfstdt then cmstdy=cmendt-rfstdt;
else if cmendt ge rfstdt then cmstdy=(cmendt-rfstdt)+1;
end;

length epoch$ 40;
if nmiss (cmstdt,rfstdt,rfendt)=0 then do;
if cmstdt<rfstdt and cmstdt ne. then epoch='SCREENING';
else if cmstdt ge rfstdt and cmstdt le rfendt then epoch='TREATMENT';
else if cmstdt gt rfendt and rfendt=. then epoch='FOLLOW-UP';
end;
run;

/*Create CMSEQ*/
%sort(ds=cm_dm, var=usubjid);

data cmseq;
set cm_dm;
by usubjid;
if first.usubjid then cmseq=1;
else cmseq+1;
run;

*****CMSPID: CRF page number

/*derive cmsprep*/
data cmpresp;
set cmseq;
if cmtrt in ("Atenolol" "Advil" "centrum" "Ancef" "Nexium" "Demerol" "Heparin" "Doxycyline" "Prednisone" "Benadryl" "Ibuprofen" "Imodium" "Reglan" "Methadone""Oxycodone" "Advair" "Hydrocodone " "Milk of Magnesia" "Tylenol" "Vitamin D/Calcium" " Taxol") then cmpresp="Y";
if cmpresp="Y" and cmtrt in ("Atenolol" "Advil" "Centrum" "prednisone" "Benadryl" "Ibuprofen" "tylenol" "Vitamin D/Calcium" "Doxycycline")
cmoccur='Y';
else if cmpresp="Y" and cmtrt in ("Ancef" "Nexium" "Demerol" "Heparin" "reglan" "imodium") then cmoccur="N";
else if cmpresp="Y" and cmtrt in ("methadone" "milk of magnesia" "Adavir" "Hydrocodone") then cmoccur=" ";
if cmpresp="Y" and cmoccur=" " then cmstat="Not done";
else cmstat=" ";
if cmstat="Not done" then cmreasnd="forgot to collect";
run;

data sdt.sdtm_cm;
set cmpresp;
run;

/*supplemental qualifiers----------suppqual
studyid------------study identifier
rdomain------------related domain
idvar--------------Identify variable
idvarval-----------Identifier variable value
qnam---------------Name of the variable 
qlabel-------------label
qval---------------variable value
qeval---------------evaluated
qorgin-------------origin of the data or source of the data*/

data supp;
set raw.nsmed;
usubjid=catx('-', study, stdysite, patient); 
keep usubjid atc:;
run;

data cm;
set sdt.sdtm_cm;
keep studyid usubjid domain cmseq;
run;

/*Merge supp to cm*/
Proc sort data=supp;
by usubjid;
run;
proc sort data=cm;
by usubjid;
run;

data supp_cm;
merge supp cm;
by usubjid;
run;

data supp1 (drop=domain cmseq);
set supp_Cm;
rdomain=domain;
idvar="CMSEQ";
idvarval=cmseq;
run;

/*transpose the data*/
proc sort data=suppq;
by studyid usubjid rdomain idvar idvarval;
run;
proc transpose data=suppq out=supp2;
var ATC:;
by studyid usubjid rdomain idvar idvarval;
run;

data supp3 (drop=_Name_ _Label_ _col1);
set supp2;
qnam=_Name_;
qlabel=_Label_';
qval=col1;
qeval="investigator";
qorig="Who dictionary";
run;

data sdt.supp_cm;
set supp3;
run;

SUbdm: Pull the values for race, other race related information, raw random dataset





