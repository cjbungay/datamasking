![SAP HANA Academy](https://yt3.ggpht.com/-BHsLGUIJDb0/AAAAAAAAAAI/AAAAAAAAAVo/6_d1oarRr8g/s100-mo-c-c0xffffffff-rj-k-no/photo.jpg)
# SAP HANA Security #
### Tutorial Video Playlist ### 
[SAP HANA Security](https://www.youtube.com/playlist?list=PLkzo92owKnVz2TJuTO9B71U7gTsG6beVJ)

## Data Masking ##
Data masking provides an additional layer of object-level access control, for example to complement SELECT privileges. Regular object-level access control privileges return either a result set or an error: insufficient privileges. With data masking, a result set is always returned. However, unless the user has the UNMASKED object-level privilege, the data will be hidden as defined by the mask.

Different implementations are possible. Below a simple example.
### Tutorial Video ### 
[![What's New](https://img.youtube.com/vi/JBQ4bMCRXSY/0.jpg)](https://youtu.be/JBQ4bMCRXSY " [2.0 SPS 04] SAP HANA Service, Security, Data Masking (CF) - SAP HANA Academy")

![Data Masking](https://help.sap.com/doc/PRODUCTION/18956b5b1b004347b7c350f9378bd2e3/Cloud/en-US/loioac83d82af853469c9e83b6c67d432c70_LowRes.png)

## Code Sample ##
In this example for data masking, you execute the following statements to create three users.
```
CREATE USER mask_owner PASSWORD Password1234 NO FORCE_FIRST_PASSWORD_CHANGE;
CREATE USER data_owner PASSWORD Password1234 NO FORCE_FIRST_PASSWORD_CHANGE;
CREATE USER end_user PASSWORD Password1234 NO FORCE_FIRST_PASSWORD_CHANGE;
```
Connect as data_owner and create a table called credit_tab and populate it with sensitive data.
```
CONNECT data_owner PASSWORD Password1234;
CREATE ROW TABLE credit_tab (Name VARCHAR(20), CREDIT_CARD VARCHAR(19));
INSERT INTO credit_tab VALUES ('John', '1111-1111-1111-1111');
INSERT INTO credit_tab VALUES ('James', '2222-2222-2222-2222');
```
Connect as mask_owner and create the user-defined function credit_mask and grant execute permission on the function to data_owner.
```
CONNECT mask_owner PASSWORD Password1234;
CREATE FUNCTION credit_mask(INPUT VARCHAR(19))
RETURNS OUTPUT VARCHAR(19) LANGUAGE SQLSCRIPT AS
    temp VARCHAR(19);
BEGIN
     SELECT LEFT(INPUT,4) || '-XXXX-XXXX-' || RIGHT(INPUT,4) INTO temp FROM SYS.DUMMY;
     OUTPUT := temp;
END;
GRANT EXECUTE ON credit_mask TO data_owner;
```
Connect as data_owner and create the view credit_view using the credit_mask function as the masking expression.
```	
CONNECT data_owner PASSWORD Password1234;
CREATE VIEW credit_view AS SELECT * FROM credit_tab
WITH MASK (CREDIT_CARD USING mask_owner.credit_mask(credit_card));
GRANT SELECT ON credit_view TO end_user;	
```
Connect as end_user and query credit_view.
```	
CONNECT end_user PASSWORD Password1234;
SELECT * FROM data_owner.credit_view;
```
Connect as data_owner and alter the masking definition for credit_view.
```	
CONNECT data_owner PASSWORD Password1234;
SELECT * FROM data_owner.credit_view;
ALTER VIEW credit_view AS SELECT name, credit_card FROM credit_tab 
WITH MASK (NAME USING 'AAAA', CREDIT_CARD USING 'XXXX');
```
Connect as end_user and query credit_view.
```	
CONNECT end_user PASSWORD Password1234;
SELECT * FROM data_owner.credit_view;
```
Connect as data_owner and drop masking for the name and credit_card columns.
```	
CONNECT data_owner PASSWORD Password1234;
ALTER VIEW credit_view DROP MASK (NAME, CREDIT_CARD);
```
Connect as end_user and query credit_view. Because data masking has been dropped, all data contained in the view is now visible.
```
CONNECT end_user PASSWORD Password1234;
SELECT * FROM data_owner.credit_view;
```
Connect as data_owner and alter the masking information on the credit card to once again use the credit_mask function.
```
CONNECT data_owner PASSWORD Password1234;
ALTER VIEW credit_view ADD MASK (CREDIT_CARD USING mask_owner.credit_mask(credit_card));
```
Connect as end_user and query credit_view.
```
CONNECT end_user PASSWORD Password1234;
SELECT * FROM data_owner.credit_view;
```
Connect as data_owner and grant unmasked privilege on object to end_user.
```	
CONNECT data_owner PASSWORD Password1234;
GRANT UNMASKED ON credit_view TO end_user;
```
Connect as end_user and query credit_view.
```
CONNECT end_user PASSWORD Password1234;
SELECT * FROM data_owner.credit_view;
```
Connect as data_owner and revoke unmasked privilege on object to end_user.
```	
CONNECT data_owner PASSWORD Password1234;
REVOKE UNMASKED ON credit_view FROM end_user;
```

### Documentation ### 
* [Data Masking – SAP HANA Security Guide for SAP HANA Service](https://help.sap.com/viewer/18956b5b1b004347b7c350f9378bd2e3/Cloud/en-US/aaa8d28740ea4cfd907d5a70017b1633.html)
* [CREATE VIEW Statement (Data Definition) – SAP HANA SQL and System Views Reference for SAP HANA Service](https://help.sap.com/viewer/7c78579ce9b14a669c1f3295b0d8ca16/Cloud/en-US/20d5fa9b75191014a33eee92692f1702.html)

