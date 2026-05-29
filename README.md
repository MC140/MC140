- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

BlankMeasure = 
VAR SelectedID = SELECTEDVALUE('perso_tam9730 cust_dim'[cust_intrl_id])
RETURN
IF(
    NOT ISBLANK(SelectedID),
    CALCULATE(
        SUM('perso_tam9730 MRA4_refined_transactions'[trxn_base_am])
    )
    -- Returns BLANK automatically when no ID selected
)

ShowPromptMessage = 
IF(
    ISBLANK(SELECTEDVALUE('perso_tam9730 cust_dim'[cust_intrl_id])),
    "Please enter a Customer Global ID",
    BLANK()
)


-- For the TABLE visual visibility
ShowTable = 
IF(NOT ISBLANK(SELECTEDVALUE('perso_tam9730 cust_dim'[cust_intrl_id])), 1, 0)

-- For the CARD/MESSAGE visual visibility  
ShowMessage = 
IF(ISBLANK(SELECTEDVALUE('perso_tam9730 cust_dim'[cust_intrl_id])), 1, 0)


