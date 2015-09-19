<?php
	#Read file content to array
	// $filepath = './log/sample.log';
	$filepath = 'c://xampp/htdocs/playlab/log/sample.log';
	$text = file_get_contents($filepath);
	$group = explode("\n", $text);

	$filepath_analyze_log = 'c://xampp/htdocs/playlab/log/loganalyze.log';
	// $filepath_analyze_log = './log/loganalyze.log';
	$keyword = array(
		'func' => 'api/users',
		'cpm' => 'count_pending_messages',
		'gm' => 'get_messages',
		'gfp' => 'get_friends_progress',
		'gfs' => 'get_friends_score',
		'status' => 'status=',
		'service' => 'service=',
		'connect' => 'connect=',
		'dyno' => 'dyno='
		);
	$no_of_call = $mean = $median = $mode = $dyno = array();	
	$mean_array = $mode_array = $median_array = $dyno_array = array();
	$endpoint = $log_analyze = '';

	#list of endpoint
	$endpoint_array = array(
		'count_pending_messages',
		'get_messages',
		'get_friends_progress',
		'get_friends_score',
		'get',
		'post'
		);

	#Search keyword for analyze
	foreach ($group as $gvalue) {
		$find_func = strpos($gvalue, $keyword['func']); //api/users
		#Check each line have keyword
		if($find_func !== false) {
			$find_method = strpos($gvalue, 'GET');
			
			if($find_method !== false) { // Method = GET
				$find_cpm = strpos($gvalue, $keyword['cpm']);
				$find_gm = strpos($gvalue, $keyword['gm']);
				$find_gfp = strpos($gvalue, $keyword['gfp']);
				$find_gfs = strpos($gvalue, $keyword['gfs']);
				if($find_cpm !== false) { //count_pending_messages
					$endpoint = $keyword['cpm'];
				} elseif ($find_gm !== false) { //get_messages
					$endpoint = $keyword['gm'];
				} elseif ($find_gfp !== false) { //get_friends_progress
					$endpoint = $keyword['gfp'];
				} elseif ($find_gfs !== false) { //get_friends_score
					$endpoint = $keyword['gfs'];
				} else {
					$endpoint = 'get';
				}
			} else { // Method = POST
				$endpoint = 'post';
			} //End if

			$find_status = strpos($gvalue, $keyword['status']);
			$find_service = strpos($gvalue, $keyword['service']);
			$start = $find_service+strlen($keyword['service']); //start position for substr
			$service_time = substr($gvalue, $start,$find_status-($start+3)); // 3 for 'ms' and ' ' between service and status
			
			$find_connect = strpos($gvalue, $keyword['connect']);
			$start = $find_connect+strlen($keyword['connect']); //start position for substr
			$connect_time = substr($gvalue, $start,$find_service-($start+3)); // 3 for 'ms' and ' ' between connect and service
			
			#Summary of response time
			$response_time = $connect_time + $service_time;
			if(isset($mean[$endpoint])) {
				$mean[$endpoint] += $response_time;
			} else {
				$mean[$endpoint] = $response_time;
			}
			
			#Count dupplicate response time Ex. $mode_array[33] = 10 -> response time 33 sec 10 times
			if(isset($mode_array[$endpoint][$response_time]))			
				$mode_array[$endpoint][$response_time] += 1;
			else
				$mode_array[$endpoint][$response_time] = 1;

			#Store response time for sort
			$median_array[$endpoint][] = $response_time;

			$find_dyno = strpos($gvalue, $keyword['dyno']);
			$start = $find_dyno+strlen($keyword['dyno']); //start position for substr
			$dyno_name = substr($gvalue, $start,$find_connect-($start+1)); // +1 for ' ' between dyno and connect
			
			#Count dupplicate dyno Ex. $dyno_array[web.9] = 10 (times)
			if(isset($dyno_array[$endpoint][$dyno_name]))			
				$dyno_array[$endpoint][$dyno_name] += 1;
			else
				$dyno_array[$endpoint][$dyno_name] = 1;			
			
			#Count number of request
			if(isset($no_of_call[$endpoint])) {
				$no_of_call[$endpoint]++;
			} else {
				$no_of_call[$endpoint] = 1;
			}
			
		} //End if
	} //End foreach

	#Analyze by endpoint
	foreach ($endpoint_array as $evalue) {
		$mean[$evalue] = $mean[$evalue]/$no_of_call[$evalue];

		#Sort Lowest value -> highest value with new key
		sort($median_array[$evalue]);

		#Sort Lowest key -> highest key
		ksort($mode_array[$evalue]); 

		#Search for too much of dupplicate value
		$mode[$evalue] = array_search(max($mode_array[$evalue]), $mode_array[$evalue]); 
		$num_of_array = count($median_array[$evalue]); 
		$odd_number = $num_of_array%2 == 0;

		#find median follow odd or even of number of array
		if($odd_number) {
			$half_of_array = $num_of_array/2;
			$median[$evalue] = $median_array[$evalue][$half_of_array-1] + $median_array[$evalue][$half_of_array]; 
			$median[$evalue] = $median[$evalue]/2;
		} else {
			$half_of_array = ceil($num_of_array/2);
			$median[$evalue] = $median_array[$evalue][$half_of_array-1];
		}
		#Search for too much of dupplicate value
		$dyno[$evalue] = array_search(max($dyno_array[$evalue]), $dyno_array[$evalue]); 

		echo 'Endpoint : '.$evalue.'<br>';
		echo 'The number of times the URL was called : '.$no_of_call[$evalue].' times<br>';
		echo 'The median of the response time : '.$median[$evalue].' ms<br>';
		echo 'The mode of the response time : '.$mode[$evalue].' ms<br>';
		echo 'The mean of the response time : '.$mean[$evalue].' ms<br>';
		echo 'The "dyno" that responded the most : '.$dyno[$evalue].'<br>'.'<br>';

		$log_analyze .= 'Endpoint : '.$evalue.PHP_EOL;
		$log_analyze .= 'The number of times the URL was called : '.$no_of_call[$evalue].' times'.PHP_EOL;
		$log_analyze .= 'The median of the response time : '.$median[$evalue].' ms'.PHP_EOL ;
		$log_analyze .= 'The mode of the response time : '.$mode[$evalue].' ms'.PHP_EOL ;
		$log_analyze .= 'The mean of the response time : '.$mean[$evalue].' ms'.PHP_EOL ;
		$log_analyze .= 'The "dyno" that responded the most : '.$dyno[$evalue].PHP_EOL .PHP_EOL ;
	}

	$fp = fopen($filepath_analyze_log, 'w');
	fwrite($fp, $log_analyze);
	fclose($fp);	

?>
