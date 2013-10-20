---
layout: post
title: php 显示cgminer当前部分状态
categories:
- 未分类
tags: []
published: true
date: '2013-09-21'
permalink: /2013/php-display-cgminer-status
---
	<?php
	#
	# Sample Socket I/O to CGMiner API
	#
	date_default_timezone_set('Asia/Chongqing');
	function getsock($addr, $port)
	{
	 $socket = null;
	 $socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
	 if ($socket === false || $socket === null)
	 {
		$error = socket_strerror(socket_last_error());
		$msg = "socket create(TCP) failed";
		echo "ERR: $msg '$error'\n";
		return null;
	 }
	 
	 $res = socket_connect($socket, $addr, $port);
	 if ($res === false)
	 {
		$error = socket_strerror(socket_last_error());
		$msg = "socket connect($addr,$port) failed";
		echo "ERR: $msg '$error'\n";
		socket_close($socket);
		return null;
	 }
	 return $socket;
	}
	#
	# Slow ...
	function readsockline($socket)
	{
	 $line = '';
	 while (true)
	 {
		$byte = socket_read($socket, 1);
		if ($byte === false || $byte === '')
			break;
		if ($byte === "&#92;&#48;")
			break;
		$line .= $byte;
	 }
	 return $line;
	}
	#
	function request($cmd)
	{
	 $socket = getsock('127.0.0.1', 4028);
	 if ($socket != null)
	 {
		socket_write($socket, $cmd, strlen($cmd));
		$line = readsockline($socket);
		socket_close($socket);
	 
		if (strlen($line) == 0)
		{
			echo "WARN: '$cmd' returned nothing\n";
			return $line;
		}
	 
		#print "$cmd returned '$line'\n";

		if (substr($line,0,1) == '{')
			return json_decode($line, true);
	}
	 
	 return null;
	}
	#
	 $r = request("{\"command\":\"summary\",\"parameter\":\"\"}");
	 $status = $r['STATUS'][0];
	 $when = $status['When'];
	 
	 $description = $status['Description'];
	 $summary = $r['SUMMARY'][0];
	 $speed = $summary['MHS av'];
	 $elspsed = $summary['Elapsed'];
	 $found_blocks = $summary['Found Blocks'];
	 $accepted = $summary['Accepted'];
	 $rejected = $summary['Rejected'];
	 $hardware_err = $summary['Hardware Errors'];
	 $utility = $summary['Utility'];
	 $discarded = $summary['Discarded'];
	 $stale = $summary['Stale'];
	 $net_block = $summary['Network Blocks'];
	 $total_mh = $summary['Total MH'];
	 $work_ulti = $summary['Work Utility'];
	 $best_share = $summary['Best Share'];
	 $energy = $elspsed*0.5*$utility/1000/3600;
	 
	?>
	 
	<html>
	<head>
		<link rel="icon" href="./favicon.ico" type="image/x-icon" />
		<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
		<style type="text/css">
			table,td,th
			{
				border:1px solid black;
			}
			h1 {font-family:verdana,arial,sans; font-size:15pt; color:blue;}
			td { text-align:center;font-family:verdana,arial,sans; font-size:13pt; color:blue; background:#ecffff;}
			td.two { font-family:verdana,arial,sans; font-size:13pt; color:blue; background:#ecffff; }
			td.h { font-family:verdana,arial,sans; font-size:13pt; color:blue; background:#c4ffff; }
			td.err { font-family:verdana,arial,sans; font-size:13pt; color:black; background:#ff3050; }
			td.warn { font-family:verdana,arial,sans; font-size:13pt; color:black; background:#ffb050; }
			td.sta { font-family:verdana,arial,sans; font-size:13pt; color:green; }
			td.tot { font-family:verdana,arial,sans; font-size:13pt; color:blue; background:#fff8f2; }
			td.lst { font-family:verdana,arial,sans; font-size:13pt; color:blue; background:#ffffdd; }
			td.hi { font-family:verdana,arial,sans; font-size:13pt; color:blue; background:#f6ffff; }
			td.lo { font-family:verdana,arial,sans; font-size:13pt; color:blue; background:#deffff; }
	 
		</style>
		<title>ZLY's miner's status - <?php echo $description;?> </title>
		<script>
			// var url = "https://www.btcguild.com/api.php?api_key=50bd432d484ac778128f2cc98621d0f3";
			// var xmlHttp = null;
	 
			// xmlHttp = new XMLHttpRequest();
			// xmlHttp.open( "GET", url, false );
			// xmlHttp.send( null );
			// xmlHttp.setRequestHeader("access-control-allow-origin","*");
			// xmlHttp.setOrigin("https://www.btcguild.com/");
			// var resp = xmlHttp.responseText;
			// var unpaidBTC = JSON.parse(resp).user.unpaid_rewards;
			// document.getElementById("unpaid_rewards").value = unpaidBTC;
			// alert(unpaidBTC);
	 
		</script>
	</head>
	 
	<body>
	<center>
	<h1> Miner Status:</h1>
	<table border="1">
	 
	<tr>
		<td>Average Speed</td>
		<td><?php echo $speed;?> MH/s</td>
	</tr>
	 
	<tr>
		<td>Running Time</td>
		<td><?php echo round(floatval($elspsed)/60/60/24,2);?> Days</td>
	</tr>
	 
	<tr>
		<td>Hardware Errors</td>
		<td><?php echo $hardware_err;?></td>
	</tr>
	 
	<tr>
		<td>Energy Cost</td>
		<td><?php echo round($energy,2);?> kWh</td>
	</tr>
	 
	<tr>
		<td>When</td>
		<td><?php echo date('m-d G:i:s',$when);?></td>
	</tr>
	 
	<tr>
		<td>Found Blocks</td>
		<td><?php echo $found_blocks;?></td>
	</tr>
	 
	<tr>
		<td>Accepted</td>
		<td><?php echo $accepted;?></td>
	</tr>
	 
	<tr>
		<td>Rejected</td>
		<td><?php echo $rejected;?></td>
	</tr>
	 
	<tr>
		<td>Utility</td>
		<td><?php echo $utility;?></td>
	</tr>
	 
	<tr>
		<td>Discarded</td>
		<td><?php echo $discarded;?></td>
	</tr>
	 
	<tr>
		<td>Stale</td>
		<td><?php echo $stale;?></td>
	</tr>
	 
	<tr>
		<td>Network Blocks</td>
		<td><?php echo $net_block;?></td>
	</tr>
	 
	<tr>
		<td>Total TH</td>
		<td><?php echo round(floatval($total_mh)/1000/1000,3);?></td>
	</tr>
	 
	<tr>
		<td>Best Share</td>
		<td><?php echo $best_share;?></td>
	</tr>
	 
	</table>
	</center>
	</body>
	</html>