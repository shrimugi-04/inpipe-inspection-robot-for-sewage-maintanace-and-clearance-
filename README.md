# inpipe-inspection-robot-for-sewage-maintanace-and-clearance-
human assisted robot to monitor the status inside the drainges and main sewer lines and also to clearthe sludges and wastes blocking the flow of wastes
module used for the monitoring of the status of the drainages 
#include <Wire.h>
#include <ESP8266WiFi.h>
#include "nodemcu_arduino_pins.h"
#include "iot_functions.h"

const char* host = "steamspace.in";

unsigned long lastExecutionTime = 0;

const int MOISTURE_SENSOR_PIN = A0;
const int GAS_SENSOR_PIN = D5;

int moisture;

String gas_status;

void setup() {
  Serial.begin(9600);
  delay(10);
  
  pinMode(GAS_SENSOR_PIN, INPUT);

  initWiFi("PROJECT", "withlove", 1);
  
  requestURL(host, "/projects/ser24218_drainage_alert_system/setVariables.php?keywords>connection_status=1");
}

void loop() 
{
  // Send sensor data to the server every 10 seconds
  if(millis() - lastExecutionTime >= 8000) 
  {
    lastExecutionTime = millis();
	
	moisture = analogRead(MOISTURE_SENSOR_PIN) / 8;

	if(moisture > 100)
		moisture = 100;
	
	if(!digitalRead(GAS_SENSOR_PIN))
	{
		gas_status = "NOT%20SAFE";
		delay(300);
	}
	else
	{
		gas_status = "SAFE";
		delay(300);
	}

    String url = "/projects/ser24218_drainage_alert_system/setVariables.php?keywords>refresh=1&moisture=";
    url += String(moisture);
	url += "&gas_status=";
    url += gas_status;
    
    requestURL(host, url);
  }
  
  delay(10); 
}






<?php
	include('session.php');
?>
<!DOCTYPE html>
<html >

<head>
	<meta charset="UTF-8">
	<title>Drainage Blockage Alert in IoT</title>	
	
	<link rel="stylesheet" href="css/reset.min.css">
	<link rel="stylesheet" href="css/style.php?theme=purple">
	<link rel="stylesheet" href="css/modular.css">
	
	<!-------------------------------------
	//Auto Reload Page using AJAX 
	-------------------------------------->
	<script src="js/jquery.min.js"></script>
	<script src="js/ajax-functions.js"></script>
	
	<script type="text/javascript"> 
	function delay(ms){
	   var start = new Date().getTime();
	   var end = start;
	   while(end < start + ms) {
		 end = new Date().getTime();
	  }
	}
	$(document).ready(function() 
	{
		function functionToLoadFile() 
		{
			jQuery.get('getVariables.php?keywords>refresh=1', function(data) 
			{
				if (data == "1") 
				{
					document.getElementById('buzzer').play();
					delay(1000);
					$(location).attr('href', 'app.php');
					jQuery.get('setVariables.php?keywords>refresh=0');
				}
				setTimeout(functionToLoadFile, 5000);
			});
		}

		setTimeout(functionToLoadFile, 10);
	});

	</script>
	
	<!-------------------------------------
	//Auto Reload Page using AJAX - END
	-------------------------------------->
	
</head>

<body>
	<div class="cta" style="float: right; padding-top: 7px;">
		<span class="button-small">Welcome <?= ucfirst($_SESSION['normal_user']). " "; ?><em><a href="logout.php">Logout</a></em></span>
	</div>
	<div class="pen-title">
		<h1>Drainage Blockage Alert in IoT</h1>
	</div>
	<audio id="buzzer"><source src="beep.ogg" type="audio/ogg"></audio>
	<div class="form-module form-module-medium">
		<ul class="top-menu">
			<li><a class="active" href="app.php">Home</a></li>
			<li class="fr"><a class="side-menu" href="editDB.php" target="_blank"><img src="images/edit_db.png">Edit DB</a></li>
			<li class="fr"><a class="side-menu" href="viewDB.php" target="_blank"><img src="images/database.png">View DB</a></li>
		</ul>
		
		<div class="content">
			<h2 class="headings">Drainage Parameters</h2>
			<?php
			
				if(get_default_value('moisture') < 10)
				{
					$moisture_css = "green";
					$blockage = "No Blockage";
				}
				else
				{
					$moisture_css = "red";
					$blockage = "Blocked";
				}
			
				if(get_default_value('gas_status') == "SAFE")
				{
					$gas_status_css = "green";
				}
				else
				{
					$gas_status_css = "red";
				}
				
				echo "<div style='float: left; margin: 0 30px 30px;'><h2>Moisture</h2> <br/> <br/>";
				echo "<a class='button-large $moisture_css'><img src='images/moisture.png' alt=''>" . get_default_value('moisture') . "</a></div>";
				
				echo "<div style='float: left; margin: 0 30px 30px;'><h2>Gas Status</h2> <br/> <br/>";
				echo "<a class='button-large $gas_status_css'><img src='images/dust.png' alt=''>" . get_default_value('gas_status') . "</a></div>";
				
				echo "<div style='clear: both;'><br><br><br></div>";
				
				echo "<div style='width: 400px; margin: 0 30px 30px;'><br/> <br/>";
				echo "<a style='display: inline-block; width: 210px; margin: 0 30px 30px;' class='button-large $moisture_css'><img src='images/caution.png' alt=''>" . $blockage . "</a></div>";

			?>
			<div style="clear: both;"> </div>
			<br/>
		</div>
				
	<script src='js/jquery.min.js'></script>

</body>
</html>
