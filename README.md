## sap 접속 및 getTable

-java list객체 저장 및 사용

우선 sap 담당자에게 제공받은 파일을 확인한다. 

sapjco3.jar 는 라이브러리로 등록.
 
sapjco3.dll 은 
C:\Windows\System32 <=여기에 복사.
 
 
 
```bash
package sapTest;
 
import java.util.concurrent.CountDownLatch;
import java.io.File;
import java.io.FileOutputStream;
import java.util.Properties;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
 
import com.sap.conn.jco.AbapException;
import com.sap.conn.jco.JCoContext;
import com.sap.conn.jco.JCoDestination;
import com.sap.conn.jco.JCoDestinationManager;
import com.sap.conn.jco.JCoException;
import com.sap.conn.jco.JCoField;
import com.sap.conn.jco.JCoFunction;
import com.sap.conn.jco.JCoFunctionTemplate;
import com.sap.conn.jco.JCoStructure;
import com.sap.conn.jco.JCoTable;
import com.sap.conn.jco.ext.*;
 
public class main {
 
 
           List<Map<String, Object>> list =null;  //조회된 데이터를 담을 리스트
 
     static String ABAP_AS = "ABAP_AS_WITHOUT_POOL";  //sap 연결명(연결파일명으로 사용됨)
 
 
     //sap 연결파일 생성
     static void createDestinationDataFile(String destinationName, Properties connectProperties)
     {
         File destCfg = new File(destinationName+".jcoDestination");
         
         if(!destCfg.exists()){
         try
         {
             FileOutputStream fos = new FileOutputStream(destCfg, false);             
             connectProperties.store(fos, "for tests only !");
             fos.close();
         }
         catch (Exception e)
         {
             throw new RuntimeException("Unable to create the destination files", e);
         }
         }
     }
     
     
     public static void getTableTest() throws JCoException
     {
        System.out.println("테이블가져오기 실행");
        JCoDestination destination = JCoDestinationManager.getDestination(ABAP_AS);
              
               //연결정보확인.
               System.out.println("Attributes:");
         System.out.println(destination.getAttributes());
         System.out.println();
 
              //리모트 펑션(?) 암튼 펑션명으로 호출
         JCoFunction function = destination.getRepository().getFunction("SAP_DATA"); 
         if(function == null)
             throw new RuntimeException("SAP_DATA not found in SAP.");
 
         try
         { 
             function.execute(destination);
             System.out.println("실행완료::!!");
         }
         catch(AbapException e)
         {
             System.out.println(e.toString());
             return;
         }
 
         //펑션에서 테이블 호출
         JCoTable codes = function.getTableParameterList().getTable("테이블명");
 
              list = new ArrayList<Map<String, Object>>(); 
        
            //루프돌면서 데이터 조회
             for (int i = 0; i < codes.getNumRows(); i++) 
         {
           codes.setRow(i);    
                Map<String, Object> map = new HashMap<String, Object>();
 
                map.put("컬럼1", codes.getString("컬럼1"));
                map.put("컬럼2", codes.getString("컬럼2"));
                map.put("컬럼3", codes.getString("컬럼3"));
                
                //리스트에 담아서 사용
               list.add(map);
 
         }
 
     }
         
     
 public static void main(String[] args) throws JCoException {
 // TODO Auto-generated method stub
 System.out.println("시작");
 
         //연결프로퍼티 생성
         Properties connectProperties = new Properties();
 
     connectProperties.setProperty(DestinationDataProvider.JCO_ASHOST, "SAP IP정보");  //SAP 호스트 정보
     connectProperties.setProperty(DestinationDataProvider.JCO_SYSNR,  "00");           //인스턴스번호
     connectProperties.setProperty(DestinationDataProvider.JCO_CLIENT, "100");         //SAP 클라이언트
     connectProperties.setProperty(DestinationDataProvider.JCO_USER,   "USER");   //SAP유저명
     connectProperties.setProperty(DestinationDataProvider.JCO_PASSWD, "PASS");  //SAP 패스워드
     connectProperties.setProperty(DestinationDataProvider.JCO_LANG,   "EN");       //언어
         
         //프로퍼티를 이용하여 연결파일을 생성. 
        //실행되고 있는 응용시스템 경로에 생성됨.
         createDestinationDataFile(ABAP_AS, connectProperties);
 
 
        getTableTest();
        
      
 }
 
}
```
