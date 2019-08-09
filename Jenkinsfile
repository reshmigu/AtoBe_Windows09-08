
@Grapes(
    @Grab(group='org.jsoup', module='jsoup', version='1.6.2')
)
import org.jsoup.*
import org.jsoup.nodes.*
import org.jsoup.select.*
import java.util.*;
import java.text.SimpleDateFormat
//import hudson.plugins.git.*
import hudson.Util;


node (label: 'windows'){
  
def totalpassed = 0
def totalfailed = 0
def totalnontexecuted = 0
def totaltests = 0
def date = new Date()
def date_format=new SimpleDateFormat("EEE dd MMM HH:mm:ss z")
def build_date =date_format.format(date) 

def gitBranch='master'
List testArray = new ArrayList<Map<String,String>>();
    withMaven(maven:'maven') {
      env.gitBranch=gitBranch
	  env.Mode="${params.modes}"
	try{
			
        stage('Checkout') {
             git url: 'https://github.com/reshmigu/AtoBe_ver1.3.git', credentialsId: 'master', branch: 'master'
        }

		stage('Build') {
            bat 'mvn clean package shade:shade'
            def pom = readMavenPom file:'pom.xml'
            env.version = pom.version			
        }

        stage('Image') {
                bat 'docker stop restassured || exit 0'
				bat 'docker rm restassured || exit 0'
                cmd = "docker rmi restassured:${env.version} || exit 0"
                bat cmd
                bat "docker build -t restassured:${env.version} ."
            
        }

        stage ('Run') {

       		 print "${params}"
		 
        	 if ("${params.modes}" == "DRY_RUN") {
			 
       			 bat "docker run -p 8081:8081 -h restassured --name restassured --net host -m=500m restassured:${env.version} DRY_RUN"
      	     }
      	     else if("${params.modes}" == "RUN") {
	  	 	 	 bat "docker run -p 8081:8081 -h restassured --name restassured --net host -m=500m restassured:${env.version} RUN"
      	     }
      	     else if("${params.modes}" == "FULL_RUN") {
	  	 		 bat "docker run -p 8081:8081 -h restassured --name restassured --net host -m=500m restassured:${env.version} FULL_RUN"
      	     }
	   bat "docker cp restassured:/test-output ."
	
	
	print "artifacts creation"	
// Archive the build output artifacts.
    archiveArtifacts artifacts: 'test-output/emailable-report.html'
    print "artifacts creation ends"	


	//	try{ 	    
		

	def currentDir = new File("").getAbsolutePath()
	print "${currentDir}/jobs/${env.JOB_NAME}/builds/${env.BUILD_NUMBER}/archive/test-output/emailable-report.html"
	def file1 = new File("${currentDir}/jobs/${env.JOB_NAME}/builds/${env.BUILD_NUMBER}/archive/test-output/emailable-report.html")
		print "file exists = ${file1.exists()} "
		
		//file operation starts
		  
      
print "masterPage ${file1}"
Document doc = Jsoup.parse(file1, "UTF-8");


print "totalpassed ${totalpassed}" 
 	
    for (Element table : doc.getElementById("summary")) {
        print "table found"
        for (Element row : table.select("tr.passedeven")) {
            Elements tds = row.select("td");  
			Map<String,String> map1  = new HashMap<>();
			
            totalpassed=totalpassed+1			
			if (tds.size() == 4) {  				
			    map1.put("name", tds.get(1).text());
            }
            else if (tds.size() == 3) {                
               // print "doc ${tds.get(0).text()}"			
				map1.put("name", tds.get(0).text()); 
            }
		
		    map1.put("url", "${env.BUILD_URL}/console"); 
		    map1.put("flag", "1"); 

			testArray.add(map1);
        }
		for (Element row : table.select("tr.failedeven")) {
            Elements tds = row.select("td");  
			Map<String,String> map1  = new HashMap<>();
            totalfailed=totalfailed+1			
			if (tds.size() == 4) {    
				map1.put("name", tds.get(1).text());            
                print "test: ${tds.get(1).text()}"
            }
            else if (tds.size() == 3) {     
				map1.put("name", tds.get(0).text()); 			
                print "test: ${tds.get(0).text()} from build"
            }
			  map1.put("url", "${env.BUILD_URL}/console"); 
		    map1.put("flag", "2");
			testArray.add(map1);
        }
    }
	print "total passed ${totalpassed}"
	print "total failed ${totalfailed}"
	print "testArray ${testArray.get(0)}"
		//file operation ends
		
	
//	}catch(Exception){}
          	  
	   	
      
	}
	}catch(Exception){
	 currentBuild.result = 'FAILURE'
	}
	

    }
		

		
		stage('mail'){		    
			env.totalpassed=totalpassed
            env.totalfailed=totalfailed
			env.totalnontexecuted= totalnontexecuted
			env.totaltests=totalpassed + totalfailed + totalnontexecuted
			env.build_date = build_date.toString()
			env.build_result = currentBuild.currentResult
			env.build_duration =  Util.getTimeSpanString(System.currentTimeMillis() - currentBuild.startTimeInMillis)
			print "build_duration: ${env.build_duration}"
			//print "change sets: ${currentBuild.changeSets.items}"  
		    
		 def config = [:]
	//def subject = config.subject ? config.subject : "${env.JOB_NAME} - Build #${env.BUILD_NUMBER} - ${currentBuild.currentResult}!"
	def subject = config.subject ? config.subject : "${env.JOB_NAME} Build Test Report - ${currentBuild.currentResult}!"
       
        // Attach buildlog when the build is not successfull
        def attachLog = (config.attachLog != null) ? config.attachLog : (currentBuild.currentResult != "SUCCESS")
//	 def content = '${JELLY_SCRIPT,template="managed:Jelly2"}'
		 //def content = '${SCRIPT,template="managed:Tpm-Email-Template"}'

		 def content=createTemplate(env,testArray)
		 
         env.ForEmailPlugin = env.WORKSPACE
        emailext mimeType: 'text/html',
	attachLog :true,
	compressLog : true,
      //  body: '${FILE, path="test-output/emailable-report.html"}',
		
	body:content,		
        subject: subject,
        to: 'dhananjaya.k@thinkpalm.com,reshmi.g@thinkpalm.com'
		
		 }


}


//function to create template
def createTemplate(env,  testArray) {
    def sb = new StringBuilder()
	sb.append '<STYLE>BODY, TABLE, TD, TH, P {font-family:Verdana,Helvetica,sans serif;font-size:11px;color:black;}h1 { color:black; }h2 { color:black; } h3 { color:black; } TD.bg1 { color:white; background-color:#0000C0; font-size:120% } TD.bg2 { color:white; background-color:#4040FF; font-size:110% } TD.bg3 { color:white; background-color:#8080FF; } TD.test_passed { color:green; } TD.test_failed { color:red; } TD.test_none { color:gray; } TD.console { font-family:Courier New; }</STYLE><BODY>'

sb.append '<B style="font-size: 150%;">CI REPORT</B></BR>'
   //heading data starts
  sb.append '<table style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> <tbody><tr></tr> <tr>'
  
  //BRANCH_NAME
   sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> Branch</td> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> :</td>'   
  sb.append  '<td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"><b>'
  sb.append env.gitBranch
  sb.append '</b></td></tr>'
  
  //image  
  sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> Image</td> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> :</td>'  
  sb.append  '<td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"><b>'
  sb.append 'restassured'
  sb.append '</b></td></tr>'
  
   //version
  sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;">Image Version</td> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> :</td>'   
  sb.append  '<td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"><b>'
  sb.append env.version
  sb.append '</b></td></tr>'
  
    //Execution Mode
  sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> Execution Mode</td> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> :</td>'  
  sb.append  '<td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"><b>'
  sb.append env.Mode
  sb.append '</b></td></tr>'
  
    //build Id
    sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;">Build Id</td> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> :</td>'  
  sb.append  '<td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;">'
  sb.append('<a href="'+env.BUILD_URL+'"'+'><b>#'+env.BUILD_NUMBER+'</b></a>')  
  sb.append '</td></tr>'
  

  
    //build_result
    sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> Build Status</td> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> :</td>'  
  sb.append  '<td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"><b>'
  sb.append env.build_result
  sb.append '</b></td></tr>'
  
  //build_duration
    sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> Test Duration</td> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> :</td>'  
  sb.append  '<td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"><b>'
  sb.append env.build_duration
  sb.append '</b></td></tr>'
  
  //build_date
    sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> Date</td> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"> :</td>'  
  sb.append  '<td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;"><b>'
  sb.append env.build_date
  sb.append '</b></td></tr>'
  
  
  sb.append '<tr></tr></tbody></table>'
  //heading data ends
 
sb.append '</BR><TABLE><TR></TR></TABLE><BR/>'
    

//total test count
sb.append '<table cellpadding="5" style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;border-collapse:collapse;border:1px solid black;"> <colgroup><col span="1" style="width:80%;"><col span="1" style="width:10%;"></colgroup> <tbody>'

sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;border:1px solid black;"> <b style="font-size:100%;">Total number of tests</b></td> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;padding:10px;border:1px solid black;"> <b style="font-size:100%;">'
sb.append (env.totaltests + '</b></td></tr>')

sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;border:1px solid black;"> <b style="font-size:100%;">Number of tests passed</b></td> <td style="color:green;font-size:11px;font-family:Verdana,Helvetica,sans serif;padding:10px;border:1px solid black;"> <b style="font-size:100%;">'
sb.append (env.totalpassed + '</b></td></tr>')

sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;border:1px solid black;"> <b style="font-size:100%;">Number of tests failed</b></td> <td style="color:red;font-size:11px;font-family:Verdana,Helvetica,sans serif;padding:10px;border:1px solid black;"> <b style="font-size:100%;">'
sb.append (env.totalfailed + '</b></td></tr>')

sb.append '<tr> <td style="color:black;font-size:11px;font-family:Verdana,Helvetica,sans serif;border:1px solid black;"> <b style="font-size:100%;">Number of tests not executed</b></td> <td style="color:gray;font-size:11px;font-family:Verdana,Helvetica,sans serif;padding:10px;border:1px solid black;"> <b style="font-size:100%;">'
sb.append (env.totalnontexecuted + '</b></td></tr>')

sb.append '</tbody></table><BR/>'

// Status Report
sb.append  '<TABLE width="100%" border="1" style="border-collapse: collapse;">'
sb.append 	('<TR><TD class="bg1" colspan="2">&nbsp; <B>' +'Tests</B></TD></TR>') //+env.JOB_NAME
sb.append '</TABLE>'


sb.append '<TABLE width="100%" style="border-collapse: collapse;">'
 sb.append '<TR>'
  sb.append '<TD style="border: 1px solid black;" colspan="1" > &nbsp;&nbsp; <B style="font-size: 115%;">Test case name</B> </TD>'
 sb.append '<TD style="border: 1px solid black;" colspan="1" >&nbsp;&nbsp;  <B style="font-size: 115%;"> Result </B> </TD>'
   sb.append '</TR>'
   
   //for each loop starts
   
    for (ArrayList<Map<String,String>> item : testArray) {  
	if(item.flag == "1"){
	 sb.append '<TR><TD style="border: 1px solid black;" colspan="1" class="test_passed" >&nbsp;&nbsp;'
		 sb.append  '<B>'+ item.name +'</B>'
		 sb.append	'</TD><TD style="border: 1px solid black;" colspan="1" class="test_passed" >&nbsp;&nbsp;'		
		sb.append('<a href="'+item.url+'"'+' style="color:green"><b>PASS</b></a>');
		 sb.append	'</TD></TR>'
	}  
		if(item.flag == "2"){
	 sb.append '<TR><TD style="border: 1px solid black;" colspan="1" class="test_failed" >&nbsp;&nbsp;'
		 sb.append  '<B>'+ item.name +'</B>'
		 sb.append	'</TD><TD style="border: 1px solid black;" colspan="1" class="test_failed" >&nbsp;&nbsp;'		
		sb.append('<a href="'+item.url+'"'+' style="color:red"><b>FAIL</b></a>');
		 sb.append	'</TD></TR>'
	}  
	
}   
  //for each loop ends

	if(testArray.size == 0)
       sb.append  '<TR><TD colspan="2" style="border:1px solid black">No details available to show</TD></TR>'
	
	sb.append ('</TABLE><BR/></BODY>')
	return sb.toString()
}
