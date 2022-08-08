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

