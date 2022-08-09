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
/*Macro for sorting*/
%Macro sort (ds=, var=);
proc sort data=&ds; by &var; run;
%mend;


data cm1 (keep=studyid domain usubjid cmtrt cmmodify cmcat cmscat
set raw.nsmed (drop=studyid);
studyid=study;
domain='CM';
usubjid=catx('-', study, stdysite, patient);
cmtrt=medx;
cmmodify=medptx;
cmcat=upcase(lblstyp);
cmscat=upcase(tmttyp);
cmindc=tmtrsnx;
cmclas=atc4x;
cmclascd=atc4c;
cmdose=dose;
cmdoseu=doseu;
cmdosfrq=freq;
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



