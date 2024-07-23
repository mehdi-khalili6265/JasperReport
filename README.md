# JasperReport
Files of Jasper Report

![Picture1](https://github.com/user-attachments/assets/bd9a9a5e-c664-4ebe-a39d-c57321ca765f)

DECLARE

  v_blob                 BLOB;

  v_file_type            VARCHAR2 (25) := 'pdf';

  v_file_name            VARCHAR2 (50);

  v_vcContentDisposition VARCHAR2 (25)  := 'inline';

  v_DEPTNO NUMBER := :P254_ID; 

  v_hostname   VARCHAR2(30) := 'localhost'; 

  v_port       NUMBER       := '6060'; 

  v_username   VARCHAR2(50) := 'jasperadmin'; 

  v_password   VARCHAR2(50) := 'jasperadmin'; 

  v_jasper_string VARCHAR2(30) := v_username || ';' || v_password;

  v_login_url  VARCHAR2(100) := 'http://' || v_hostname || ':' || v_port || '/jasperserver/rest/login';

  v_report_url VARCHAR2(100) ;

BEGIN
-- Get the file name from the COR_SECRETARIAT table 
--Dieser Code ruft eine Jasper Report-Datei auf
SELECT 
CASE WHEN LATTER_ENGLISH = 'Y' THEN 'Latter_EN.'||v_file_type
WHEN LATTER_ENGLISH = 'N' THEN 'Latter_FA.'||v_file_type 
END
INTO v_file_name
FROM COR_SECRETARIAT
WHERE ID = :P254_COR_SECRETARIAT_ID;
v_report_url  := 'http://' || v_hostname || ':' || v_port || '/jasperserver/rest_v2/reports/reports/Apex_Report/' || v_file_name;
 -- log into jasper server
  v_blob := apex_web_service.make_rest_request_b(

    p_url => v_login_url,

    p_http_method => 'GET',

    p_parm_name => apex_util.string_to_table('j_username;j_password',';'),

    p_parm_value => apex_util.string_to_table(v_jasper_string,';')

  );
  -- download file the file name from the table 
  --die generierte Datei kann Download 
  --oder in einer Tabelle gespeichert warden

    v_blob := apex_web_service.make_rest_request_b(

    p_url => v_report_url,

    p_http_method => 'GET',

    p_parm_name => apex_util.string_to_table('P_ID',';'),

    p_parm_value => apex_util.string_to_table(v_DEPTNO,';')

  );

    OWA_UTIL.mime_header ('application/pdf', FALSE);  -- view your pdf file

   OWA_UTIL.MIME_HEADER( 'application/octet', FALSE ); -- download your pdf file

   HTP.p('Content-Length: ' || DBMS_LOB.GETLENGTH(v_blob));

   HTP.p('Content-Disposition: ' || v_vcContentDisposition ||'; filename="' || v_file_name || '"');

   OWA_UTIL.http_header_close;

   WPG_DOCLOAD.DOWNLOAD_FILE(v_blob);

  -- add file to table

  --APEX_APPLICATION.STOP_APEX_ENGINE;

 INSERT INTO COR_EXPORT_LATTER_DOCUMENT (FILE_NAME,DOCUMENT_LATTER,MIME_TYPE,ID_REF)
  VALUES (v_file_name, v_blob,'application/pdf',:P254_ID);
END;
![image](https://github.com/user-attachments/assets/ba167a55-4ec4-4a0e-9967-4e625dd43fb1)

