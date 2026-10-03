<?php



	public function dataseek(){
		
		$this->load->view("cdn");
	$this->load->view("navheader");
	  $rdate = $this->CModel->get_latest('racedate');
		$rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
      
	$crn = $this->CModel->get_latest('current_rn');
		$crv = $this->CModel->get_latest('venue');#only two char ST/HV/S1234

$gr=$this->mongo_db->select(['venue','result'])->where(['racedate' => $this->mongo_db->in([$rdate,$rdate1]),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->get('hkjc_traindata');
$rd_list=$this->mongo_db->select(['venue','racedate'])->sort('racedate', 'desc')->get('hkjc_traindata');
$rd_opt=[];
foreach($rd_list as $r)
	$rd_opt["{$r['racedate']}|{$r['venue']}"]="{$r['racedate']} | {$r['venue']}";
$data["rd_list"]=$rd_list;
		$data=$this->fetch_one_race($rdate,$crn,$crv,"live");#

		if(is_null($data['crv']))
			$data=$this->fetch_one_race($rdate1,$crn,$gr[$crn-1]['venue'],"live");#
		$data["live_history"]="live";
		$data["venue_list"]=implode("|",array_column($gr,'venue'));
		$data['title']="時光機";
		$data['is_admin']=$this->tank_auth->is_admin();

		$data["rd_opt"]=$rd_opt;
			$this->load->view("system_layout",$data);
	}
	
	public function change_time(){
		
		   $rd = $this->input->post("rd");
		   $rn= $this->input->post("rn");
			$loop_rv = $this->input->post("rv");
			#first is minstogo 
			$loop_rv=substr($loop_rv,0, 2);
			$t1 = $this->input->post("t1");#latest near -1
			
			$t2 = $this->input->post("t2");#sc or m3 or m0

			 $t1_odd= $this->mongo_db->where(['racedate' => $rdate,'venue'=>$this->mongo_db->in(["{$loop_rv}TURF", "{$loop_rv}AWT",$loop_rv]), 'raceno' => intval($rn), 'minstogo' => intval($t1)])->sort('inserttime', $this->input->post("t1s"))->getOne('hkjc_rawodd')[0];
			  $t2_odd= $this->mongo_db->where(['racedate' => $rdate,'venue'=>$this->mongo_db->in(["{$loop_rv}TURF", "{$loop_rv}AWT",$loop_rv]), 'raceno' => intval($rn), 'minstogo' => intval($t2)])->sort('inserttime', $this->input->post("t2s"))->getOne('hkjc_rawodd')[0];

			$r['t1']=$t1_odd;
			$r['t2']=$t2_odd;
			echo json_encode($r);
	}
	
	
	
	
	public function clearbu(){#when mins 3
		#ini_set('display_errors', '1');
	

		$ird=(isset($crd))?$crd:$this->CModel->get_latest('racedate');
		$crn=intval($this->CModel->get_latest('current_rn'));
		$crv=($this->CModel->get_latest('venue'));
		print_r((['racedate' => $ird, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])]));
		$data=$this->mongo_db->select(['d','biguser'])->where(['racedate' => $ird, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->getOne('hkjc_traindata')[0];
		$this->mongo_db->set(['biguser'=>''])->where(['racedate' => $ird, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->update('hkjc_traindata');
		#reset m3 ball
		foreach(range(1,count($data['d'])) as $hn){
		$this->mongo_db->set(['d.'.($hn-1).'.m0' => 0 ])->where(['racedate' => $ird, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->update('hkjc_traindata');
		$this->mongo_db->set(['d.'.($hn-1).'.isbu' => 0 ])->where(['racedate' => $ird, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->update('hkjc_traindata');
		}
		
	}
	public function cleartosc(){#when mins 3
		#ini_set('display_errors', '1');
	
	foreach(range(1,14) as $hn){
	$tablerow[]=['minstogo'=>0,'tw'=> 0,'jw'=> 0,  'h'=> $hn, 'amount'=> 0, 'od'=>0,'nd'=>0, 'amountcls'=>0,'jwcls'=>"",'twcls'=>""];
	foreach(range(-1,16) as $mins){
		
	foreach(['w','q','qp','dbld'] as $ty){
	
	$d2[$hn][$mins][$ty]=0;
	}
	}
		}
		$ird=(isset($crd))?$crd:$this->CModel->get_latest('racedate');
		$crn=intval($this->CModel->get_latest('current_rn'));
		
		$this->mongo_db->set(['biguser'=>''])->where(['racedate' => $ird, 'raceno' => intval($crn)])->update('hkjc_traindata');
		#reset m3 ball
		#foreach(range(1,14) as $hn)
		#$this->mongo_db->set(['d.'.($hn-1).'.m3' => 0 ])->where(['racedate' => $ird, 'raceno' => intval($crn)])->update('hkjc_traindata');
		$this->mongo_db->set(['d'=>$d2])->where([ 'racedate' => $ird,'raceno' => intval($crn)])->update('hkjc_wqqp');
		
	}
		public function fixscm()
 {
	#ini_set('display_errors', '1');
	 	 $rdate = $this->CModel->get_latest("racedate");
     $crn=10;
	  $crv=$this->CModel->get_latest("venue");

	 	$data=$this->mongo_db->select(['tl'])->where(['racedate'=>$this->mongo_db->in([$rdate]),'raceno'=>($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->getOne('hkjc_chart')[0]['tl'];
			
				foreach($data as $key => $t){
				    $parts=explode("####",$t["data"]);
				    $mins= intval($parts[0]);
	
					 if($mins==17){
						 $m_arr=explode('|',$parts[5]);
						# print_r($parts[8]);# [8] hn,[5] m
						 foreach(explode('|',$parts[8]) as $hni=>$hn){
							echo "$hn@ {$m_arr[$hni]} <br>";
			
				 $this->mongo_db->set(['d.'.($hn-1).'.scm' => ($m_arr[$hni]) ])->where(['racedate' => $rdate, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->update('hkjc_traindata');
				  
						 }
						 break;
				  }
					
				  
				}
	 
 }
	public function fixdbl12a()
 {
	#ini_set('display_errors', '1');
	 $rdate = $this->CModel->get_latest("racedate");
     $crn=intval($this->CModel->get_latest('current_rn'));
     $crn = $crn > 0 ? $crn : 1;
	 $loop_rv=$this->CModel->get_latest("venue");
	# print_r(["racedate" => $rdate, "raceno" => $crn,'venue'=>$loop_rv]);
	 $tdata = $this->mongo_db
         ->select(["d"])
         ->where(['racedate' => $rdate, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$loop_rv}TURF", "{$loop_rv}AWT",$loop_rv])])
         ->getOne("hkjc_traindata")[0]["d"];
	$last1234= $this->mongo_db
         ->select(["result"])
         ->where(['racedate' => $rdate, 'raceno' => intval($crn)-1,'venue'=>$this->mongo_db->in(["{$loop_rv}TURF", "{$loop_rv}AWT",$loop_rv])])
         ->getOne("hkjc_traindata")[0]["result"];
		 $last_odd= $this->mongo_db->where(['racedate' => $rdate,'venue'=>$this->mongo_db->in(["{$loop_rv}TURF", "{$loop_rv}AWT",$loop_rv]), 'raceno' => intval($crn)-1, 'minstogo' => intval(-1)])->sort('inserttime', 'desc')->getOne('hkjc_rawodd')[0];
	
		#	var_dump($last1234);
		$lastresult=explode(',',$last1234);

		// --- FIRST LOOP ---
        foreach ($last_odd['DBL'] as $h => $odd) {
            $parts = explode('-', $h);
            $leg1 = (int)$parts[0];
            $leg2 = (int)$parts[1];
            $index = $leg2 - 1;

            try {
                // Initialize default values for the row if they don't exist
                if (!isset($tdata[$index])) {
                    $tdata[$index] = [];
                }

                if ($leg1 == $lastresult[0]) {
                    $c1 = round($odd / $last_odd['WIN'][$lastresult[0]], 1);
                    $tdata[$index]['dbl1'] = $c1;
					$this->mongo_db->set(['d.'.($index).'.dbl1' => ($c1) ])->where(['racedate' => $rdate, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$loop_rv}TURF", "{$loop_rv}AWT",$loop_rv])])->update('hkjc_traindata');
                }
                
                if ($leg1 == $lastresult[1]) {
                    $c2 = round($odd / $last_odd['WIN'][$lastresult[1]], 1);
                    $tdata[$index]['dbl2'] = $c2;
						$this->mongo_db->set(['d.'.($index).'.dbl2' => ($c2) ])->where(['racedate' => $rdate, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$loop_rv}TURF", "{$loop_rv}AWT",$loop_rv])])->update('hkjc_traindata');
                }
            } catch (Throwable $e) {
                // Catching all errors/exceptions to match Python's bare except:
                $tdata[$index]['dbl1'] = 0;
                $tdata[$index]['dbl2'] = 0;
            }
        }
		
	#var_dump($tdata);

        // --- SECOND LOOP ---
        $grouped_dbl = [];
        foreach ($last_odd['DBL'] as $th => $to) {
            $th_parts = explode('-', $th);
            $leg2 = isset($th_parts[1]) ? (int)$th_parts[1] : -1;
            if ($leg2 !== -1) {
                $grouped_dbl[$leg2][] = $to;
            }
        }

        // --- SECOND LOOP (NOW LIGHTNING FAST) ---
        $max_runner = count( $tdata)+ 1;
        for ($h = 1; $h <= $max_runner; $h++) {
            try {
                // Fetch pre-grouped odds for runner $h instantly
                $c = $grouped_dbl[$h] ?? [];
                if (empty($c)) {
                    continue; 
                }
                
                $sum_c =$last_odd['DBLHV'][(string)$h] ?? null;
                if (!$sum_c) {
                    continue; // Skip division by zero if sum_c doesn't exist
                }

                $w_values = [];
                foreach ($c as $to) {
                    // Quick numeric check
                    if ($to == 0 || !is_finite($to) || is_nan($to)) {
                        continue; 
                    }

                    // Consolidated math to reduce variable allocations
                    // 1 / (to / 0.825) simplifies mathematically to 0.825 / to
                    $one_over_c = 0.825 / $to;
                    $x_val = (10000 * $one_over_c) / $sum_c;
                    $w_values[] = $to * $x_val;
                }

                $index = $h - 1;
                if (!isset($tdata[$index])) {
                   $tdata[$index] = [];
                }

                if (!empty($w_values)) {
                  
					  $tdata[$index]['dbla'] = round(min($w_values) / 1000, 1);
						$this->mongo_db->set(['d.'.($index).'.dbla' => (round(min($w_values) / 1000, 1)) ])->where(['racedate' => $rdate, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$loop_rv}TURF", "{$loop_rv}AWT",$loop_rv])])->update('hkjc_traindata');
                } else {
                    $tdata[$index]['dbla'] = 0; 
                }

            } catch (Exception $e) {
                echo 'dbla ' . $e->getMessage() . PHP_EOL;
            }
        }
    }
		
	     
                
 
	public function get_maxmup($tdata)
 {
	// $hn=["10"];
	  foreach ($tdata['d'] as $v) {
		  if($v['maxmup_has']==1 and $v['hn']!='1')
         $hn[] = $v['hn'];
     }
	 return implode('、',$hn);
 }
	public function overbet_m()
 {
     //ini_set('display_errors', '1');
     $rdate = $this->CModel->get_latest("racedate");
     $crn = intval($this->CModel->get_latest("current_rn"));
     $crn = $crn > 0 ? $crn : 1;
     $tdata = $this->mongo_db
         ->select(["d"])
         ->where(["racedate" => $rdate, "raceno" => $crn])
         ->getOne("hkjc_traindata")[0]["d"];

     foreach ($tdata as $v) {
         $unsort_hn[] = [
             "hn" => $v["hn"],
             "scw" => floatval($v["scw"]),
             "w" => floatval($v["w"]),
             "m" => $v["m"],
             "scm" => $v["scm"],
         ];
     }
     usort($unsort_hn, function ($a, $b) {
         if ($a["w"] == $b["w"]) {
             //brand_order are same
             return $a["m"] <=> $b["m"]; //sort by title
         }
         return $a["w"] <=> $b["w"];
     });

     $mdata = array_reduce(
         $unsort_hn,
         function ($carry, $item) {
             $carry[] = $item["m"];
             return $carry;
         },
         []
     );

     $hn = [];

     $r = [];
     for ($i = count($mdata) - 1; $i >= 0; $i--) {
         $compare_arr = array_slice($mdata, 0, $i);
         for ($j = 0; $j <= count($compare_arr) - 1; $j++) {
             if ($mdata[$i] >= $compare_arr[$j]) {
                 $r[] = $compare_arr[$j];
             }
         }

         if (count($r) >= 2) {
             #echo $unsort_hn[$i]['hn'],"@",implode(",",$r),'<br>';#hn
             $hn[] = $unsort_hn[$i]["hn"];
         }
         $r = [];
     }
     return $hn;
 }
	public function mobile()
{
    #ini_set('display_errors', '1');
    $d = $this->input->post("did");
    $skip_str = "異常";
    $newstring = substr($d, -4);
    if (!in_array($newstring, ["2fe0", "b2e8"])) {
        //	echo '';
        //	return 1;
    }
    $rdate = $this->CModel->get_latest("racedate");
    $crn = intval($this->CModel->get_latest("current_rn"));
    $crn = $crn > 0 ? $crn : 1;
    $json["alertonoff"] = "off";
    $dt = $this->mongo_db
        ->select(["posttime"])
        ->where(["racedate" => $rdate, "raceno" => $crn])
        ->get("hkjc_traindata");
    foreach ($dt as $vn => $v) {
        $v = $v["posttime"];
        $start = new DateTime($v);

        $end = new DateTime(date("Y-m-d H:i"));

        $d = $end->diff($start);
        $day = $d->format("%r%d");
        $h = $d->format("%r%H");
        $m = $d->format("%r%i");

        $remind_mins = 1440 * $day + 60 * $h + $m - 1;
        $remind_mins = $remind_mins > 0 ? $remind_mins : 0;
    }
    if ($remind_mins <= 5) {
        $json["alertonoff"] = "on";
    }
    $cold_choice = [];
    $abnormal2 = [];
    $abnormal1 = $this->overbet_m(); #higher 2x
    $data = $this->mongo_db
        ->select(["d", "biguser"])
        ->where(["racedate" => $rdate, "raceno" => $crn])
        ->getOne("hkjc_traindata")[0];
    $range = $data["biguser"];
    foreach ($data["d"] as $r) {
        if ($r["m3up"] >= 2 and $r["w"] >= 10) {
            #冷門爆波

            $cold_choice[] = $r["hn"];
        }

        if (($r["qoverbet"] >= 1 or $r["qpoverbet"] >= 1) and $r["m3up"] >= 2) {
            #紋身
            $abnormal2[] = $r["hn"];
        }
    }

    $time = date("H:i:s", strtotime("now"));
    $json["title"] = "$time~ 第 $crn 場 ($remind_mins 分鐘開跑)";
    $abnormal = array_merge($abnormal1, $abnormal2);

    $abnormal = array_merge($cold_choice, $abnormal);

    if (count($abnormal) > 4) {
        $skip_str = "亂局太多異常";
    }
    $hn = implode(",", $abnormal);
    $json["reminder"] = "[$skip_str] $hn \n [大戶範圍] $range";

    echo json_encode($json);
}
	
public function loadcsv(){
	$this->load->library('CSVReader');
	$csvData = $this->csvreader->parse_file('jockey_trainer_stat.csv'); //path to csv file
	$d=[];
	foreach ($csvData as $v) 
    {
		
   $d[($v['j'].$v['t'])]=['w_match' => $v['w_match'] ,'p_match' =>$v['p_match'] ,'total' => $v['total'] ,'wc' => $v['wc'] ,'pc' => $v['pc']];
    }
	return ($d);
	
}
public function loadmfhist() {
	# ini_set('display_errors', '1');
	 $this->load->driver('cache',
        array('adapter' => 'apc', 'backup' => 'file', 'key_prefix' => 'racetl_')
);
    $d = json_decode($this->input->raw_input_stream, true);
    if (!$d) {
        echo json_encode(null);
        return;
    }

    $crd = $d["raceDate"] ?? '';
    $crn = intval($d["raceNo"] ?? 1);
    $crv = substr($d["venue"] ?? '', 0, 2);
    $dt  = intval($d["time"]) ?? '';

    // 1. Create a unique cache key based on the query parameters
    $cacheKey = "race_chart_{$crd}_{$crn}_{$crv}";

    // 2. Attempt to fetch data from the cache first cache data real time update 
    $data = $this->cache->get($cacheKey);

    if (!$data) {
        // 3. Cache Miss: Query MongoDB
        $result = $this->mongo_db->select(['tl'])
            ->where([
                'racedate' => $crd,
                'raceno'   => $crn,
                'venue'    => $this->mongo_db->in(["{$crv}TURF", "{$crv}AWT", $crv])
            ])
            ->getOne('hkjc_chart');

      $data = $result[0]['tl'] ?? $result['tl'] ?? [];

        // 4. Save to cache for 60 minutes (3600 seconds) so the next request is instant
        if (!empty($data)) {
            $this->cache->save($cacheKey, $data, 5);
        }
    }
#var_dump( $data);
    // 5. Fast linear scan on the cached data
    $match = array_find($data, function ($item) use ($dt) {
        return isset($item['time']) && ($item['time']== $dt);
    });

    echo json_encode($match['data'] ?? null);
}

public function loadmft_live($crd,$crn,$crv){#return latest 1 record

				
					#check data 0 mins times and add last character
				$data=$this->mongo_db->select(['tl_live'])->where(['racedate'=>$this->mongo_db->in($crd),'raceno'=>intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->getOne('hkjc_chart')[0]['tl_live'];
 if($data){
				 $parts=explode("####",$data["data"]);
				
				   foreach(range(1,5) as $l)
				array_unshift($parts , '');
				    
				    $data0=implode("@@@@",$parts);
				    return $data0;
				 }
				  return "";

}
public function loadmft($crd,$crn,$crv){#return timestamp 0mins many other mins one

					$count0=1;
					#check data 0 mins times and add last character
				$data=$this->mongo_db->select(['tl'])->where(['racedate'=>$this->mongo_db->in($crd),'raceno'=>intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->getOne('hkjc_chart')[0]['tl'];
				$data30=[];
				$data0=[];
					$dblu_mins=0;
				foreach($data as $key => $t){
				    $parts=explode("####",$t["data"]);
				    $mins= intval($parts[0]);
					if($dblu_mins==0){
					$dblu_=array_sum(explode("|",$parts[13]));#check cutoff mins
					$dblu_2=array_sum(explode("|",explode("####",$data[$key+1]["data"])[13]));
					if($dblu_==$dblu_2 and $mins<30)
						$dblu_mins=$mins;
					}
					 if($mins<=40 and $mins!=-1){
				    $data30[$mins]=['time'=>$t['time'],'data'=>$t["data"]];#last mins graph save
				    }
					
				    if($mins<=0){

				    $data0[]=['time'=>$t['time'],'data'=>$t["data"]];
				    }
				}
				$data30=array_values($data30);
		        $result = array_merge( $data30,  $data0);
		      #add live 
				$tls=array_column($result,'time');#last is 0 mins 

$r=implode(",",$tls);

return [$r,$dblu_mins];
}

	public function loadresult(){
		#ini_set('display_errors', '1');
		$rd=$this->input->post("rd");
		$ird=date('Ymd',strtotime($rd));
		$irds=date('Y/m/d',strtotime($rd));
		$irn=(int)$this->input->post("rn");
$irv=$this->input->post("rv");
$irv=substr($irv,0,2);

if(!in_array($irv,['ST',"HV"])){
	
	$url="https://iosbsinfo02.hkjc.com/infoRN/IOSBS/HR05_GetInfo.ashx?QT=HR_RESULTS&Html=1&MeetingId={$ird}{$irv}&Race=$irn&Lang=zh-HK";

#var_dump($url);
	$xml_string =print_r(jcweb_go($url), true);#
	
 function DOMinnerHTML(DOMNode $element) 
{ 
    $innerHTML = ""; 

	  $innerHTML = $element->ownerDocument->saveHTML($element);
	$innerHTML=str_replace("result_table", "dividendTb", $innerHTML);
    return $innerHTML; 
}


$html =new DOMDocument();
$html->preserveWhiteSpace = false;
$html->formatOutput       =true;
$html ->loadHTML($xml_string);
$finder = new DomXPath($html);
$nodes = $finder->query("//table[@id='result_dividends_table']");

foreach ($nodes as $rn=>$node) 
    {
		
    echo DOMinnerHTML($node);
    }
return 1;
}


function DOMinnerHTML(DOMNode $element) 
{ 
    $innerHTML = ""; 
    $children  = $element->childNodes;

    foreach ($children as $child) 
    { 
        $innerHTML .= $element->ownerDocument->saveHTML($child);
    }
$innerHTML=str_replace("派彩", "", $innerHTML);
$innerHTML=str_replace("/racing/", "https://racing.hkjc.com/racing/", $innerHTML);
    return $innerHTML; 
} 

$url="https://racing.hkjc.com/racing/information/Chinese/Racing/ResultsAll.aspx?RaceDate=$irds";

do{
	$xml_string =print_r(jcweb_go($url), true);#
	
	if(strlen($xml_string)>0)
		break;
}while(1);
#var_dump($url);
$html =new DOMDocument();
$html->preserveWhiteSpace = false;
$html->formatOutput       = true;
$html ->loadHTML($xml_string);
$finder = new DomXPath($html);
$classname="f_fr";
$nodes = $finder->query("//div[contains(concat(' ', normalize-space(@class), ' '), ' $classname ')]");

foreach ($nodes as $rn=>$node) 
    {
		if($rn==($irn-1))
    echo DOMinnerHTML($node);
    }
}


public function sb2($arr=null)//api handle win/dbl   win sb
    {
	#ini_set('display_errors', '1');
		#$input_amt=8800;
		$oversea_mode=0;
		$crv=$this->input->post('venue');
		$crv2=$this->CModel->get_latest('venue');
		$crv3=$this->input->post('raceVenue');
	

		$crv4=substr($crv3,0,2);
		foreach (['qin'=>'q','win'=>'w','dbl'=>'dbl','qpl'=>'qp','fct'=>'fct'] as $amt_k =>$amt_v){
		if(!empty($this->input->post($amt_k.'amt'))){
			$input_panel=$amt_v;
			$input_amt=intval($this->input->post($amt_k.'amt'));
			break;
		}
		
		}
	
		if(!empty($this->input->post('qinamt')) and !empty($this->input->post('qplamt'))){
			$input_panel='qqp';
			$input_amt=intval($this->input->post('qinamt'));
		}

		$backtest=$this->input->post('bt');
		if($backtest){
			foreach (['qin'=>'q','qpl'=>'qp'] as $amt_k =>$amt_v){
		if(!empty($this->input->post($amt_k.'Amt'))){
			$input_panel=$amt_v;
			$input_amt=intval($this->input->post($amt_k.'Amt'));
			break;
			}}

			
		}
		$tp=$this->input->post('type');
						
		$crd=$this->input->post('raceDate');
		$ird=(isset($crd))?$crd:$this->CModel->get_latest('racedate');
		$crn=intval($this->input->post('rn'))+intval($this->input->post('raceNo'));
		$refresh_call=false;
			if(!empty($this->input->post('mqinval')))
		$refresh_call=true;
		$token=$this->input->post('did');
		$input_pcblt=(int)$this->input->post('pc');
		if(!is_null($arr)){//call func from refresh  refresh_data
			
			$crn=intval($arr['rn']);
			$ird=$this->CModel->get_latest('racedate');
			#$runner_count=intval($arr['runner_count']);
			$input_bet_amount=intval($arr['qinamt']);
			$input_banker=$arr['qinb'];
			$input_leg=$arr['qinl'];
			$input_panel="q";
			$token=$this->input->post('did');
			$uid=$arr['uid'];
			$input_pcblt=$arr['pc'];
		

		}else{#real time 
		
header('Access-Control-Allow-Origin: *');header('Access-Control-Allow-Headers: *'); 
$have_ball=[];

$input_q_qp_banker=$this->input->post('qinb').$this->input->post('qinBanker').$this->input->post('qplb');

if($this->input->post('qinb')==$this->input->post('qplb'))
$input_q_qp_banker=$this->input->post('qinb').$this->input->post('qinBanker');

$input_bet_amount=intval($this->input->post('amt'))+$input_amt.$this->input->post('fctamt');

$input_banker=$this->input->post('banker').$input_q_qp_banker.$this->input->post('fctb');

$input_leg=$this->input->post('win')."|".$this->input->post('Leg')."|".$this->input->post('qinl')."|".$this->input->post('qinLeg')."|".$this->input->post('qpll')."|".$this->input->post('fctl');

$input_q_qp_legs=$this->input->post('Leg')."|".$this->input->post('qinl')."|".$this->input->post('qinLeg')."|".$this->input->post('qpll');
if($this->input->post('qinl')==$this->input->post('qpll'))
	$input_q_qp_legs=$this->input->post('Leg')."|".$this->input->post('qinl')."|".$this->input->post('qinLeg');
$input_leg=$input_leg.$input_q_qp_legs;

if($crn<1)
		$crn=intval($this->CModel->get_latest('current_rn'))-1;
	$crn=intval($this->input->post('rn'))+intval($this->input->post('raceNo'));
	if(!is_null($this->input->post('scrd'))){#oversea
		$oversea_mode=1;
		$crv=$this->input->post('v');
		$crd=$this->input->post('scrd');
		$crv2=$crv;
		$crv3=$crv;
		$crv4=$crv;
		$ird=$crd;
	}
	$ird=date('Y-m-d');
	$rdate1=date('Y-m-d', strtotime("$crd +1 day"));
	/*$crd="2026-09-23";
	$ird=$crd;
	$crv="HV";
	#$crn=1;*/
	if($oversea_mode){
		
		
		$ird=$crd;
	}
		$runner_count=0;	
#var_dump(['racedate' =>$this->mongo_db->in([$ird,$rdate1,$crd]), 'raceno' => $crn,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2,$crv3])]);
	#$input_panel=$this->input->post('panel');
	$data=$this->mongo_db->where(['racedate' =>$this->mongo_db->in([$ird,$rdate1,$crd]), 'raceno' => $crn,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2,$crv3])])->get('hkjc_traindata')[0]['d'];

	foreach ($data as $r) {
     if ($r["w"] > 0) {//SCR 
        
    $runner_count+=1;
    } 
	
    if ($r["m3up"] > 0) {//click only when 0 mins
        
        $have_ball[] = $r["hn"];
    } 

	 if ($r["m0up"] > 0 and $r["m0"] > 0) {//click only when 0 mins
        
        $have_ball[] = $r["hn"];
    } 
}

		
		}
	#print_r($have_ball);
$have_ball=array_unique($have_ball);		
$w_string = "";
$q_string = "";
$qp_string = "";
$fct_string = "";
$dbl_string = "";
	
		
$banker = $input_banker;
$DBL_mode=false;
if(!is_null($this->input->post('dbl1'))){#is DBL mode
	

	$buy_banker=$this->input->post('dbl1');
	$buy_legs=$this->input->post('dbl2');
	$dbl_rx=explode("|", $buy_banker);
	$dbl_rx1=explode("|", $buy_legs);
	$DBL_mode=true;
	
}

if(strpos($input_leg,'@')){#is DBL mode
	
	$tmp=explode("@", $input_leg);
	$buy_banker=explode("|", $tmp[0]);
	$buy_legs=explode("|", $tmp[1]);
	$dbl_rx=explode("|", $tmp[0]);
	$dbl_rx1=explode("|", $tmp[1]);
	$DBL_mode=true;
	
}
if(!$backtest){
	if (!empty($banker)) {// Real time
    $buy_banker = explode(",", $banker);
} else {
    $buy_banker = null;
}
$leg_with = $input_leg;
$buy_legs = explode("|", $leg_with);
	
}else{//backtest vs real time

$banker = $input_banker;
if (!empty($banker)) {
    $buy_banker = explode(",", $banker);
} else {
    $buy_banker = null;
}
$leg_with = $input_leg; 
$buy_legs = explode(",", $leg_with);

}
$buy_legs=array_filter(array_unique($buy_legs));
#print_r($buy_legs);
$buy_comb = sampling($buy_banker, $buy_legs, 2);

$result = $this->mongo_db
    ->sort("inserttime", "desc")
    ->select(['WIN',"QIN", "QPL", "fct",'DBL'])
    ->where(["racedate" => $this->mongo_db->in([$ird,$rdate1,$crd]), "raceno" => (int)($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2,$crv4])])
    ->getOne("hkjc_rawodd");
$x=$this->users->get_user_by_did($token);
$this->mongo_db->set('refresh_data',$this->input->post(NULL, TRUE))
    ->where('uid', (int)$x->user_id)
		->update('hkjc_user');
if(is_null($x))
$sbconfig =$this->loadqbconfig($uid);
else
$sbconfig =$this->loadqbconfig($x->user_id);
$r = $this->mongo_db->row_array($result);
$raw_win = $r["WIN"];
$raw_q = $r["QIN"];
$raw_fct = $r["fct"];
$raw_qp = $r["QPL"];
$raw_dbl = $r["DBL"];

if(isset($raw_dbl))
foreach ($raw_dbl as $legs => $odd) {
	
    list($L1, $L2) = explode("-", $legs);
	$leg_key=intval($L1)."-".intval($L2);
    $dbl_obj[$leg_key] = $odd;
		#$leg_key2=swap_change($leg_key);
		# $dbl_obj[$leg_key2] = $odd;
}
foreach ($raw_q as $legs => $odd) {
    list($L1, $L2) = explode("-", $legs);
	$leg_key=intval($L1)."-".intval($L2);
	#$leg_key2=swap_change($leg_key);
    $q_obj[$leg_key] = $odd;
}
foreach ($raw_qp as $legs => $odd) {
    list($L1, $L2) = explode("-", $legs);
	$leg_key=intval($L1)."-".intval($L2);
	#$leg_key2=swap_change($leg_key);
    $qp_obj[$leg_key] = $odd;
	
}
$fct_obj = [];
foreach ($raw_fct as $legs => $odd) {
    list($L1, $L2) = explode("-", $legs);
	$leg_key=intval($L1)."-".intval($L2);
	$leg_key2=swap_change($leg_key);
    $fct_obj[$leg_key] = $odd;
	$fct_obj[$leg_key2] = $odd;
}
 $params=[$input_bet_amount,$banker,$leg_with,['W'=>$sbconfig['wr'],'Q'=>$sbconfig['qr'],'QP'=>$sbconfig['qpr'],'Q_autopcblt'=>$input_pcblt],$have_ball];

$this->load->library('smartbet',$params,'QST');
$this->load->library('smartbet',$params,'QPST');
 $this->load->library('smartbet',$params,'FST');
 $this->load->library('smartbet',$params,'SMST');
$this->load->library('smartbet',$params,'WST');
#999 odd change to est cal 

if(isset($raw_dbl))
$this->load->library('smartbet',$params,'DBLST');

	if($DBL_mode){
		
	foreach ($dbl_rx as $bk1)
	foreach ($dbl_rx1 as $bl1)
	
	if($dbl_obj[$bk1."-".$bl1]>0){
		$result2 = $this->mongo_db
    ->sort("inserttime", "desc")
    ->select(['WIN'])
    ->where(["racedate" => $this->mongo_db->in([$ird,$rdate1,$crd]), "raceno" => (int)($crn)+1,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2,$crv4])])
    ->getOne("hkjc_rawodd");
$r2 = $this->mongo_db->row_array($result2);
$nextr_win_odd = $r2["WIN"];
		#999
		if($dbl_obj[$bk1."-".$bl1]>=999){
		$new999odd=round($raw_win[$bk1]*$nextr_win_odd[$bl1]*0.825,-1);
		#$new999odd=999;
		$this->DBLST->add_datarow([$bk1." - ".$bl1,$new999odd,0,0,"D"],$dbl_rx);
		}else
		$this->DBLST->add_datarow([$bk1." - ".$bl1,$dbl_obj[$bk1."-".$bl1],0,0,"D"],$dbl_rx);
    
	}
	
	
	
	
	}
	
foreach ($buy_legs  as $leg_key)
if(!empty($leg_key))
 $this->WST->add_datarow([$leg_key,$raw_win[$leg_key],0,0,"W"]);
foreach ($buy_comb as $i=>$leg_key) {
	if(empty($leg_key))
		continue;
	
	$leg_key2=swap_change($leg_key);
	$odd22 = $fct_obj[$leg_key2];
	$odd2 = $fct_obj[$leg_key];
	#var_dump($leg_key);
    if (array_key_exists($leg_key, $q_obj)) {
		$odd = $q_obj[$leg_key];
if(!$odd)
		continue;
     $this->QST->add_datarow([str_replace("-"," - ",$leg_key),$odd,0,0,"Q"],$banker);
	//check here ?
	 $this->SMST->add_smdatarow2([str_replace("-"," - ",$leg_key),$odd,1000,$odd*1000,"Q"],$banker);
	 if($odd2==0)
		 $odd2=1;
	  if($odd22==0)
		 $odd22=1;

	$sum_d=(1/$odd2+1/$odd22);#zero 
	 $this->SMST->add_smdatarow2([str_replace("-"," - ",$leg_key),$odd2,1000*(1/$odd2)/$sum_d,1000*(1/$odd2)/$sum_d*$odd2,"F"]);#1-2
	 $this->SMST->add_smdatarow2([str_replace("-"," - ",$leg_key),$odd22,1000*(1/$odd22)/$sum_d,1000*(1/$odd2)/$sum_d*$odd22,"F"]);#2-1
    }
	
	if($runner_count>6){
    if ( array_key_exists($leg_key, $qp_obj)) {#error if <6 runner
        $odd3 = $qp_obj[$leg_key];
     
		 $this->QPST->add_datarow([str_replace("-"," - ",$leg_key),$odd3,0,0,"QP"],$banker);
    }}

	 $this->FST->add_datarow([str_replace("-"," - ",$leg_key),$odd2,0,0,"F"]);
}
  $w_string="w##";
 $this->WST->cal_summary();
$w_string .= $this->WST->display_headerstring('W');
$w_string .= $this->WST->display_rowstring();
 

 $this->QST->cal_summary();

$q_string .= $this->QST->display_headerstring('Q');#not in _0x3aa2f9[0x4].split('##')[2]
$q_string .= $this->QST->display_rowstring();#show in _0x3aa2f9[0x4].split('##')[2]
$red_qin="##";
if(!$backtest and intval($input_bet_amount)>=8800)
$red_qin= $this->QST->redqin();

if($runner_count>6){
 $this->QPST->cal_summary();
$qp_string .= $this->QPST->display_headerstring('QP');
$qp_string .= $this->QPST->display_rowstring();
}

$this->FST->cal_summary();

$fct_string = $this->FST->display_headerstring('F');
$fct_string .= $this->FST->display_rowstring();
if(isset($raw_dbl)){
 $this->DBLST->cal_summary();
$dbl_string2 = $this->DBLST->display_headerstring('D');
$dbl_string3 = $this->DBLST->display_rowstring();

$dbl_string="dbl##$dbl_string2$dbl_string3";
$dbl_string=str_replace("-"," / ",$dbl_string);}
//Smart Match
$sm_string = "";


 $this->SMST->cal_smsummary();
$sm_string .= $this->SMST->display_smheaderstring();
$sm_string .= $this->SMST->display_smrowstring();

$extrahit_string= $this->QST->get_condom($banker,$raw_win,$raw_q);

#$red_qin='RED_Q@@RED_RETURNhttps://youtu.be/o_kt4vC1fSY?t=988

$check=$this->tank_auth->verify_user_tokendid($token);
if(!$check){
	echo '';
}


switch($input_panel){
	

                    case "w":
				
					#$w_string="w##0.4842|325|±123||||3333##3|90|92|3456@@7|90|92|3456";
					$return["data"]=$w_string;
					echo json_encode($return);
                       break;
                    case "p":
                       break;
                    case "wp":
                       break;
                    case "dbl":

					$return["data"]=$dbl_string;
					echo json_encode($return);
                        break;
                    case "q":
					$w_string='##';
					$qp_string='##';
					$fct_string='##';
                      break;
                    case "qp":
					$sm_string='##';
					$q_string='##';
					$fct_string='##';
                       break;
                    case "qqp":
				$w_string='##';
					$fct_string='##';
                       break;
                    case "fctb":
					$sm_string='##';
					$q_string='##';
					$qp_string='##';
                      break;
                    case "fctbm":
                    $sm_string='##';
					$q_string='##';
					$qp_string='##';
                        break;
                    case "tri":
                     break;
                    case "tce":
                        
                        break;
                    case "ff":
                       break;
                    case "qtt":
                        
                        break;

	
}


if ($backtest) {
    for ($x = 1; $x <= 2; $x++) {
        $sm_string = substr_replace($sm_string, "", -1);
    }
    switch ($tp) {
        case "q":
            $return["d"] = "$q_string######$sm_string";

            break;
        case "qp":
            $return["d"] = "####$qp_string######";
            break;
        case "qqp":
            $return["d"] = "$q_string##$qp_string##$sm_string";

            break;
    }

    echo json_encode($return);
} else {
	$cost=$input_amt*count($buy_legs);
    if (!in_array($input_panel, ["dbl","w","p"])) {
        if ($input_panel == "qp") {
            $extrahit_string = "##";
        }
		#var_dump($red_qin);
if(($input_pcblt)>0)
	$extra_pc_header="$ird%%%%%%%%%%%%%%%%%%%%%%%%%%$input_pcblt";
else
	$extra_pc_header="$ird%%$crn%%%%$cost%%$input_banker%%$input_leg%%%%%%%%%%%%%$crv%%%%%%";
	#$extra_pc_header="$ird%%%%%%%%%%%%%%%%%%%%%%%%%%";
        $return["data"] = "$extra_pc_header##$q_string
		##$qp_string
		##$fct_string
		##$sm_string

	##$red_qin
	##$extrahit_string##$w_string##$dbl_string";

        echo json_encode($return);
    }
}
	}
public function seek()
    {
		if(!$this->tank_auth->is_admin())
			redirect();
$this->load->view("cdn");
$crn=intval($this->CModel->get_latest('current_rn'));
		foreach (range(1,13) as $rno){
			$clas="";
		    $d=date('d-m-Y',strtotime($this->CModel->get_latest('racedate')));
			
			if($rno==$crn)
				$clas="background-color:#f25877";
		echo "<head><title>SEEK</title></head><div class='card' style='display:inline-block;width: 14rem;$clas'>
 
  <div  class='card-body'>
    <h5  class='card-title'>$d -- $rno</h5>
  
    <a href='https://www.moneyflow007.com/history.aspx?race=$d@$rno'>Go</a> 
  </div>
</div>";
	
    }
    }

	public function refresh(){header('Access-Control-Allow-Origin: *');header('Access-Control-Allow-Headers: *'); 
#ini_set('display_errors', '1');
$input_did=$this->input->post('did');
	$u=$this->users->get_user_by_did($input_did);
	#var_dump($u->user_id);
	$data=$this->mongo_db
    ->select(["refresh_data"])
	->where('uid', (int)$u->user_id)
    ->getOne('hkjc_user')[0]['refresh_data'];

	$data['token']=$input_did;
	$data['crd']=$this->CModel->get_latest('racedate');
	$data['crn']=$data['crn'];
	$data['uid']=$u->user_id;
	
	echo $this->sb2($data);
     

	}
	public function ballapi()
    {#ini_set('display_errors', '1');
	header('Access-Control-Allow-Origin: *');header('Access-Control-Allow-Headers: *');
	$crv=$this->CModel->get_latest('venue');#input v , rd
	$crn=intval($this->input->post("rn"));
	$tk=$this->input->post("did");//login first
	
	$have_ball=[];
	if($this->tank_auth->verify_user_tokendid($tk)){
	$data=$this->mongo_db->select(['d'])->where(['racedate'=>$this->CModel->get_latest('racedate'),'raceno'=>$crn,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv])])->getOne('hkjc_traindata')[0];

	foreach ($data["d"] as $r){
		$have_ball[]="";
		 if ($r["m0up"] > 0 or $r["m3up"] > 0)
			 $have_ball[$r["hn"]-1]=1;
		
		
	}
	
	$d['data']=$have_ball=implode("|",$have_ball);
	echo json_encode($d,JSON_UNESCAPED_UNICODE);}else{
	$d['error']='token failed';
	echo json_encode($d,JSON_UNESCAPED_UNICODE);}
	}
	public function m0api()
    {#ini_set('display_errors', '1');
	header('Access-Control-Allow-Origin: *');header('Access-Control-Allow-Headers: *');
	$crv=$this->CModel->get_latest('venue');
		$crn=intval($this->input->post("rn"));
	$tk=$this->input->post("did");//login first
	$have_ball=[];
	$have_ob=[];
	
	if($this->tank_auth->verify_user_tokendid($tk)){
	$data=$this->mongo_db->select(['d'])->where(['racedate'=>$this->CModel->get_latest('racedate'),'raceno'=>$crn,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv])])->getOne('hkjc_traindata')[0];
	foreach ($data["d"] as $r) {
    if ($r["qoverbet"] > 0 or $r["qpoverbet"] > 0) {
        $have_ob[] = 1;
    } else {
        $have_ob[] = 0;
    }
  $have_ball[]="";
		 if ($r["m0up"] > 0 or $r["m3up"] > 0)
			 $have_ball[$r["hn"]-1]=1;
}
	$have_ball=join("@",$have_ball);
	$have_ob=join("@",$have_ob);


	$ad['data']="$have_ball|$have_ob";
	echo json_encode($ad,JSON_UNESCAPED_UNICODE);}else{
	$d['error']='token failed';
	echo json_encode($d,JSON_UNESCAPED_UNICODE);}
	
	}
	public function rangeapi()
    {#ini_set('display_errors', '1');
	header('Access-Control-Allow-Origin: *');header('Access-Control-Allow-Headers: *');
	$crn=intval($this->input->post("rn"));
	$tk=$this->input->post("did");//login first
	$crv=$this->CModel->get_latest('venue');
	if($this->tank_auth->verify_user_tokendid($tk)){
	$data=$this->mongo_db->select(['biguser'])->where(['racedate'=>$this->CModel->get_latest('racedate'),'raceno'=>$crn,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv])])->getOne('hkjc_traindata')[0];
	$d['data']=str_replace(",","，",$data['biguser']);
	echo json_encode($d,JSON_UNESCAPED_UNICODE);}else{
	$d['error']='token failed';
	echo json_encode($d,JSON_UNESCAPED_UNICODE);}
	}
	public function dblapi(){
		header('Access-Control-Allow-Origin: *');header('Access-Control-Allow-Headers: *');
		$crv=$this->CModel->get_latest('venue');
		$crn=intval($this->input->post("rn"))-1;
		#scrd input v rn
	$tk=$this->input->post("did");//login first
	if($this->tank_auth->verify_user_tokendid($tk)){

	if($crn<intval($this->CModel->get_latest('total_rn'))){
	$data2=$this->mongo_db->select(['d'])->where(['racedate'=>$this->CModel->get_latest('racedate'),'raceno'=>$crn+1,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv])])->getOne('hkjc_traindata')[0];
	
	foreach($data2['d'] as $r){
		$d['dblu'][]=($r['dblu%']>=3.2)?$r['dblu%']:'';
	}
	$d['dblu']=join("|",$d['dblu']);
	}


	$ad['data']=$d['dblu'];
	echo json_encode($ad,JSON_UNESCAPED_UNICODE);}else{
	$d['error']='token failed';
	echo json_encode($d,JSON_UNESCAPED_UNICODE);}
		
		
	}
	
	
	public function scballapi(){
		header('Access-Control-Allow-Origin: *');header('Access-Control-Allow-Headers: *');
	$crv=$this->input->post("v");
	$crn=intval($this->input->post("rn"));
	$crd=$this->input->post("rd");
	$tk=$this->input->post("did");//login first

	$have_ball=[];
	if($this->tank_auth->verify_user_tokendid($tk)){
	$data=$this->mongo_db->select(['d'])->where(['racedate'=>$crd,'raceno'=>$crn,'venue'=>$this->mongo_db->in([$crv])])->getOne('hkjc_traindata')[0];

	foreach ($data["d"] as $r){
		$have_ball[]="";
		 if ($r["m0up"] > 0 or $r["m3up"] > 0)
			 $have_ball[$r["hn"]-1]=1;
		
		
	}
	

	
	$d['data']=$have_ball=implode("|",$have_ball);
	echo json_encode($d,JSON_UNESCAPED_UNICODE);}else{
	$d['error']='token failed';
	echo json_encode($d,JSON_UNESCAPED_UNICODE);}
		
	}
	public function scdblapi(){
		#ini_set('display_errors', '1');
		header('Access-Control-Allow-Origin: *');header('Access-Control-Allow-Headers: *');
	$crv=$this->input->post("v");
	$crn=intval($this->input->post("rn"));
	$crd=$this->input->post("scrd");
	$tk=$this->input->post("did");//login first
	if($this->tank_auth->verify_user_tokendid($tk)){

	
	$data=$this->mongo_db->select(['d'])->where(['racedate'=>$crd,'raceno'=>$crn,'venue'=>$this->mongo_db->in([$crv])])->getOne('hkjc_traindata')[0];
	
	foreach($data['d'] as $r){
		$d['dblu'][]=($r['dblu%']>=1)?"{$r['hn']}={$r['dblu%']}":'';#1=20|4=|3=|6=|2=|8=20|7=17|5=
	}
	$d['dblu']=join("|",$d['dblu']);
	


	$ad['data']=$d['dblu'];
	echo json_encode($ad,JSON_UNESCAPED_UNICODE);}else{
	$d['error']='token failed';
	echo json_encode($d,JSON_UNESCAPED_UNICODE);}
		
	}
	public function load_fbver(){
		
		 echo json_encode($this->mongo_db->select(['fb_ver'])->getOne('sys_var')[0]["fb_ver"]);
		
		
		
	}
	public function update_fbver(){
		$d=(float)$this->input->post("d");
		
		$this->mongo_db->set('fb_ver',$d)
		->update('sys_var');

		
	}
	public function api()#dbl smart bet here dblamt: 10 dbl1: 3 dbl2: 4|8 
    {#ini_set('display_errors', '1');
	header('Access-Control-Allow-Origin: *');header('Access-Control-Allow-Headers: *');
	$crn=intval($this->input->post("rn"));
	$tk=$this->input->post("did");//login first
	$have_ball=[];
	$have_ob=[];
	
	if($this->tank_auth->verify_user_tokendid($tk)){
	$data=$this->mongo_db->select(['d','biguser'])->where(['racedate'=>$this->CModel->get_latest('racedate'),'raceno'=>$crn])->getOne('hkjc_traindata')[0];
	if($crn<intval($this->CModel->get_latest('total_rn'))){
	$data2=$this->mongo_db->select(['d','biguser'])->where(['racedate'=>$this->CModel->get_latest('racedate'),'raceno'=>$crn+1])->getOne('hkjc_traindata')[0];
	
	foreach($data2['d'] as $r){
		$d['dblu'][]=($r['dblu%']>0)?$r['dblu%']:'';
	}
	$d['dblu']=join("|",$d['dblu']);
	}
	foreach ($data["d"] as $r) {
    if ($r["qoverbet"] > 0 or $r["qpoverbet"] > 0) {
        $have_ob[] = 1;
    } else {
        $have_ob[] = 0;
    }
    if ($r["m3up"] > 0) {//click only when 0 mins
        
        $have_ball[] = 1;
    } else {
        $have_ball[] = 0;
    }
}
	$have_ball=join("@",$have_ball);
	$have_ob=join("@",$have_ob);
	$d['ball0']="$have_ball|$have_ob";//x|x
	
	$d['range']=$data['biguser'];
	$ad['data']=$d;
	if(!empty($this->input->post("dbl1"))){
		
			$result = $this->mongo_db->sort("inserttime", "desc")->select(['DBL'])->where(["racedate" => $this->CModel->get_latest('racedate'), "raceno" => (int)($crn)])->getOne("hkjc_rawodd");
$x=$this->users->get_user_by_did($this->input->post("did"));

$sbconfig =$this->loadqbconfig($uid);

$r = $this->mongo_db->row_array($result);

$raw_dbl = $r["DBL"];
if(isset($raw_dbl))
foreach ($raw_dbl as $legs => $odd) {
    list($L1, $L2) = explode("-", $legs);
    $dbl_obj[$legs] = $odd;
}
 $params=[$this->input->post("dblamt"),'','',['W'=>$sbconfig['wr'],'Q'=>$sbconfig['qr'],'QP'=>$sbconfig['qpr'],'Q_autopcblt'=>$input_pcblt],$have_ball];
$dbl_rx=explode('|',$this->input->post("dbl1"));
$dbl_rx1=explode('|',$this->input->post("dbl2"));

if(isset($raw_dbl))
$this->load->library('smartbet',$params,'DBLST');

		
	foreach ($dbl_rx as $bk1)
	foreach ($dbl_rx1 as $bl1)
	#echo $bk1." - ".$bl1,'<br>';
	if($dbl_obj[$bk1."-".$bl1]>0){
		#odd 999 
		
		$this->DBLST->add_datarow([$bk1."-".$bl1,$dbl_obj[$bk1."-".$bl1],0,0,"D"],$dbl_rx);
    }
	
			if(isset($raw_dbl)){
				
 $this->DBLST->cal_summary();
$dbl_string2 = $this->DBLST->display_headerstring('D');
$dbl_string3 = $this->DBLST->display_rowstring();

$dbl_string="dbl##$dbl_string2$dbl_string3";
$dbl_string=str_replace("-"," / ",$dbl_string);
$d['data']=$dbl_string;
echo json_encode($d,JSON_UNESCAPED_UNICODE);
exit;
}
		}
	
	echo json_encode($ad,JSON_UNESCAPED_UNICODE);}else{
	$d['error']='token failed';
	echo json_encode($d,JSON_UNESCAPED_UNICODE);}
    }
	public function auth_login()
    {header('Access-Control-Allow-Origin: *');header('Access-Control-Allow-Headers: *');
	#ini_set('display_errors', '1');
	
	$s=[100,150,200,500];
		#$file = 'data.txt';
// Open the file to get existing content
#$current = file_get_contents($file);
#$stat= json_decode($current,true);
	$default_bet=100;
	$s=implode(',',$s);
	$p_=$this->input->post(NULL, TRUE);
	$email=$p_["email"];
	$pwd=$p_["pwd"];
	$did=(isset($p_["did"]))?$p_["did"]:'';
		
	$p=$this->tank_auth->apilogin($email,$pwd,$did);//verify login 

$d["wblt"]= "$s|$default_bet";
$d["qblt"]= "$s|$default_bet";
$d["fctblt"]= "$s|$default_bet";
$d["dblblt"]= "$s|$default_bet";
$d["tceblt"]= "$s|$default_bet";
$d["triblt"]= "$s|$default_bet";
$d["ffblt"]= "$s|$default_bet";
$d["qttblt"]= "$s|$default_bet";
$d["exblt"]= "$s|$default_bet";
$d["eng"]= "";
$d["pcblt"]= "";#new extra bet with ballball
$d["smb"]= "";
$d["msg"]= "";
$d["mfr"]= "1";
$d["mfb"]= "1";
$d["ws"]= "1";
$d["qs"]= "0";
$d["fs"]= "1";
$d["cs"]= "1";
$d["wrOn"]= "1";
$d["wr"]= "9000";
$d["prOn"]= "1";
$d["pr"]= "9000";
$d["qrOn"]= "1";
$d["qr"]= "8800";
$d["qprOn"]= "1";
$d["qpr"]= "8800";
$d["qdpOn"]= "1";
$d["qdp"]= "1";
$d["autoOn"]= "1";
$d["auto"]= "1";
$d["ver"]= (string)$this->mongo_db->select(['fb_ver'])->getOne('sys_var')[0]["fb_ver"];

#$d["rng"]= implode('|',$stat['大戶']);
	
		if($p){//Logined
	
$ld=$this->mongo_db->select(['fastbet'])->where('uid', (int)$p->id)->getOne('hkjc_user')[0]["fastbet"];


$d["smb"]= (strtotime('now')>$p->sb_pass)?'0':"1";#聰明投注on off 1 0
$d["scsmb"]=  $d["smb"];#海外聰明投注
$d["msg"]= (strtotime('now')>$p->hk_pass)?'0':"1";#範圍on off
$d["sc"]= "1";
		$hk_pass_status=(strtotime('now')>$p->hk_pass)?'已過期':date('Y-m-d',$p->hk_pass);
		$sb_pass_status=(strtotime('now')>$p->sb_pass)?'已過期':date('Y-m-d',$p->sb_pass);
		$os_pass_status=(strtotime('now')>$p->os_pass)?'已過期':date('Y-m-d',$p->os_pass);
		$id=(isset($p->id))?date('ym',$p->created).$p->id:'';
$d["id"]= $id;
#$d["email"]= $p->email;
$d["exp"]= $hk_pass_status;
$d["sexp"]= $sb_pass_status;
$d["scexp"]= $os_pass_status;
$d["scqexp"]= $sb_pass_status;// standalone later 
$d["did"]= $p->did;

foreach($ld as $k=>$v){
$d[$k]=$v;
if ($k=='ab')
$d["autobet"]= $v;
}
		echo json_encode($d,JSON_UNESCAPED_UNICODE);}else{
		
$d2["ver"]= $d["ver"];


	echo json_encode($d2,JSON_UNESCAPED_UNICODE);}
 
    }
	public function loadqbconfig($x)
    {
	$r=$this->mongo_db->select(['fastbet'])->where('uid', (int)$x)->getOne('hkjc_user')[0]["fastbet"];
	return($r);
	
 }
 public function test(){
	 return 1;
	$rd='2025-04-09';

	 print_r(date('Y-m-d',strtotime("$rd -1 days")));
 
 }
 public function saveqbconfig()
    {header('Access-Control-Allow-Origin: *');header('Access-Control-Allow-Headers: *');
	 	$d=$this->input->post(NULL, TRUE);
		$x=$this->users->get_user_by_did($d['did']);
		if($this->tank_auth->is_pass2_valid($d['did'])){
			unset($d['did']);
		$this->mongo_db->set('fastbet',$d)
    ->where('uid', (int)$x->user_id)
		->update('hkjc_user');}
	$r['data']=$this->loadqbconfig($x->user_id);
		echo json_encode($r);
	
 }
 	public function download()
    {
		return 1;
		$this->load->view("cdn");
		$this->load->view("navheader");
		$data['ver']=$this->mongo_db->select(['fb_ver'])->getOne('sys_var')[0]["fb_ver"];
        $this->load->view("download_center",$data);
    }
	public function check_horse_lastispunch($name,$rd)#one row 
    {
		
		#ini_set('display_errors', 1);
			$data["hnn"]=$name;#"上浦福旺";#	$this->input->post("hnn");
$tjcitr[]['$eq']= ['$$item.name',$data["hnn"]];
		$tjcitr[]['$lt']= ['$inserttime',strtotime("$rd -1 days")];
$data=$this->mongo_db->aggregate('hkjc_traindata', [
        ['$project' => ['_id'=>0,'racedate'=>1,'raceno'=>1,
 'items'=> [
            '$filter'=> [
               'input'=> '$d',
               'as'=> 'item',
               'cond'=> [ '$and'=> [...$tjcitr]
						]
						]
         ]
                
						]
        
    ],['$unwind'=> '$items'],[ '$sort' => ['racedate' => -1 ] ],[ '$limit' => 2]],['cursor'=>['batchSize'=>0]]);
	if(count($data)>0){
	$data=$data[0];#1 row 
	$r=[];

	$ck_col=['q%','qpl%','dbld%'];
	foreach($ck_col as $c)
	if ($data['items'][$c]>=15)
		$r[]=1;
	
	if(array_sum($r)>=1)
		return 1;
	}
		return 0;
	}
	

		public function loadxml()#The number of requests is limited to 50 per 60 minutes.
    {
			$url =XMLJSON_Link;
$xml =web_go($url);#
echo json_encode($xml ,1);
	}
		public function loadhorsehist1()#The number of requests is limited to 50 per 60 minutes.
    {
	$data["hcode"]=json_decode($this->input->raw_input_stream,1)["hcode"];
	
	
$tjcitr[]['$eq']= ['$$item.horsecode',$data["hcode"]];
		$tjcitr[]['$lt']= ['$inserttime',strtotime('-1 days')];
$data["historys"]=$this->mongo_db->aggregate('hkjc_traindata', [
        ['$project' => ['_id'=>0,'racedate'=>1,'raceno'=>1,'venue'=>1,'cls'=>1,'course'=>1,'going'=>1,'dist'=>1,
 'items'=> [
            '$filter'=> [
               'input'=> '$d',
               'as'=> 'item',
               'cond'=> [ '$and'=> [...$tjcitr]
						]
						]
         ]
                
						]
        
    ],['$unwind'=> '$items'],[ '$sort' => ['racedate' => -1 ] ],[ '$limit' => 10]],['cursor'=>['batchSize'=>0]]);
  echo json_encode($data, true);
      
    }
	public function loadhorsehist()
    {
#ini_set('display_errors', 1);
	$data["hnn"]=$this->input->post("hnn");
$tjcitr[]['$eq']= ['$$item.name',$data["hnn"]];
		$tjcitr[]['$lt']= ['$inserttime',strtotime('-1 days')];
$data["tdata"]=$this->mongo_db->aggregate('hkjc_traindata', [
        ['$project' => ['_id'=>0,'racedate'=>1,'raceno'=>1,'venue'=>1,'cls'=>1,'course'=>1,'going'=>1,'dist'=>1,
 'items'=> [
            '$filter'=> [
               'input'=> '$d',
               'as'=> 'item',
               'cond'=> [ '$and'=> [...$tjcitr]
						]
						]
         ]
                
						]
        
    ],['$unwind'=> '$items'],[ '$sort' => ['racedate' => -1 ] ],[ '$limit' => 10]],['cursor'=>['batchSize'=>0]]);
  echo json_encode($this->load->view("horsehist", $data, true));
      
    }
	function getLiveVideoID($channelId)
{
	
  /*  $videoId = null;
	$vid1="";
	$vid2="";

#https://www.youtube.com/embed/live_stream?channel=UCSuzEzGo0zIK0wcpr8j46QQ
$ch = curl_init("https://youtube.com/channel/UCaKod3X1Tn4c7Ci0iUKcvzQ/live");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'GET');
curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);

$data= curl_exec($ch);
curl_close($ch);
preg_match('/watch\?v=(.*?)\"/', (string)$data, $matches);
#https://www.youtube.com/embed/live_stream?channel=UCUdSiDWBtPZExreighMVfdQ
*/


    // Fetch the livestream page
    if($data = file_get_contents("https://youtube.com/channel/$channelId/live"))
    {
		
        // Find the video ID in there
        if(preg_match('/watch\?v=(.*?)\"/', (string)$data, $matches))
            $videoId = $matches[1];
        else
            return "";
    }
    else
        return "";

    return $videoId;
}
	public function get_url()
    {
		set_time_limit(300); 
		$d=[];
	$id=$this->input->post("d");//[]

$d=$this->getLiveVideoID($id);
	
echo json_encode($d) ;
	}
	public function fetch_one_race($rdate, $crn, $crv, $ty)
{
    $uid = $this->tank_auth->get_user_id();
    $crv2 = substr($crv, 0, 2);

    $data = [
        "race_is_pending_b30" => $this->race_is_pending(),
        "oversea_race"    => 0,
        "crd"             => $rdate,
        "crn"             => $crn,
        "crv"             => $crv,
        "crv2"            => $crv2,
        "livehistory"     => $ty,
    ];

    // 1. Handle overseas race check
    if (str_contains($crv, "S") && !str_contains($crv, "ST")) {
        $s_arr = array_map(fn($n) => "S{$n}", range(1, 8));
        $data["oversea_race"]  = 1;
        $data["oversea_raced"] = $this->CModel->get_oversearaceinfos();

        $rdate1 = date('Y-m-d', strtotime("$rdate +1 day"));
        $data["oversea_raced2"] = $this->mongo_db
            ->where([
                'racedate' => $this->mongo_db->in([$rdate, $rdate1]),
                'venue'    => $this->mongo_db->in($s_arr)
            ])
            ->sort('venue', 'asc')
            ->get('hkjc_traindata');
    }

    // 2. Fetch whole-day race data (with +1 day fallback for overnight meetings)
    $venues = ["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT", "{$crv}AWT", $crv];
    $mongo_wholedata = $this->mongo_db
        ->where([
            'racedate' => $rdate,
            'venue'    => $this->mongo_db->in($venues)
        ])
        ->get('hkjc_traindata');

    if (empty($mongo_wholedata)) {
        $rdate = date('Y-m-d', strtotime("$rdate +1 day"));
        $data["crd"] = $rdate; // Keep response date synchronized

        $mongo_wholedata = $this->mongo_db
            ->where([
                'racedate' => $rdate,
                'venue'    => $this->mongo_db->in($venues)
            ])
            ->get('hkjc_traindata');
    }

    // 3. Process whole-day metadata in-memory (eliminates extra Mongo queries)
    $total_rn = count($mongo_wholedata);
    $totalranrn = $total_rn;
    $raceinfo = [];

    foreach ($mongo_wholedata as $race) {
        $race_no = (int) ($race['raceno'] ?? 0);

        // Extract light raceinfo fields for navigation
        $raceinfo[] = [
            'racedate' => $race['racedate'] ?? null,
            'raceno'   => $race_no,
            'venue'    => $race['venue'] ?? null,
            'dist'     => $race['dist'] ?? null,
            'cls'      => $race['cls'] ?? null,
            'course'   => $race['course'] ?? null,
            'going'    => $race['going'] ?? null,
            'posttime' => $race['posttime'] ?? null,
            'result'   => $race['result'] ?? null,
        ];

        // Track live race progress
        if (empty($race['result']) && $totalranrn === $total_rn) {
            $totalranrn = $race_no - 1;
            $this->CModel->update_raceinfo('current_rn', $race_no, $crv2);
        }
    }

    // Sort raceinfo by race number ascending
    usort($raceinfo, fn($a, $b) => $a['raceno'] <=> $b['raceno']);

    // 4. Locate current targeted race document from already fetched data
    $target_rn = (int) $crn;
    $mongo_data = null;

    foreach ($mongo_wholedata as $race) {
        if ((int) ($race['raceno'] ?? 0) === $target_rn) {
            $mongo_data = $race;
            break;
        }
    }

    // 5. Fetch user configuration
    $user_config = $this->mongo_db
        ->select(['config'])
        ->where('uid', $uid)
        ->getOne('hkjc_user');

    // 6. Assemble final data array
    $data["config"]         = $user_config[0]['config'] ?? null;
    $data["totalrn"]        = $total_rn;
    $data["totalranrn"]     = $totalranrn;
    $data["raceinfo"]       = $raceinfo;
    $data["mongo_wholedata"]= $mongo_wholedata;
    $data["venueinfo"]      = $mongo_wholedata;

    $data["scr"]            = $mongo_data['scr'] ?? null;
    $data["raceresult"]     = $mongo_data['result'] ?? null;
    $data["biguser"]        = $mongo_data['biguser'] ?? null;
    $data["card"]           = $mongo_data['d'] ?? null;

    return $data;
}
	 public function vlive()
    {
	#ini_set('display_errors', '1');
		
		$uid=$this->tank_auth->get_user_id();
        $this->load->view("cdn");
		$this->load->view("navheader");
		
        $rdate = $this->CModel->get_latest('racedate');
		$rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
      
		$rdrn = $this->uri->segment(3);
		$crv2 = explode("@", $rdrn)[0];
		$crn = (int)explode("@", $rdrn)[1];
		$crv = substr($crv2 ,0,2);
		
		#var_dump($crn);
		if(is_null($rdrn)){
		$crn = $this->CModel->get_latest('current_rn');
		$crv = $this->CModel->get_latest('venue');#only two char ST/HV/S1234
			
		}

		$gr=$this->mongo_db->select(['venue','result'])->where(['racedate' => $this->mongo_db->in([$rdate,$rdate1]),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->get('hkjc_traindata');
	
	foreach ($gr as $rn=>$p){
	if(empty($p['result'])){

$crn2=$rn+1;#prepare run

break;
	}}
	#$data['ref5']=($crn>$crn2)?1:0;
	if($crn>$crn2)
		$irn=$crn;
	else
		$irn=$crn2;
		$crn=$irn;
		$data=$this->fetch_one_race($rdate,$crn,$gr[$crn-1]['venue'],"live");#
		$data["config"] = $this->mongo_db->select(['config'])->where('uid', $uid)->getOne('hkjc_user')[0]['config'];
		if(is_null($data['crv']))
			$data=$this->fetch_one_race($rdate1,$crn,$gr[$crn-1]['venue'],"live");#
		$data["live_history"]="live";
		$data["venue_list"]=implode("|",array_column($gr,'venue'));
		$data['title']="Live | $crv | $crn";
		$data['is_admin']=$this->tank_auth->is_admin();
		$data['is_lastrn']=$this->CModel->get_latest('current_rn')==$this->CModel->get_latest('total_rn');
		$data['bool_samerd']= ($rdate==date('Y-m-d'));
		if($this->tank_auth->check()){
         $this->load->view("V_layout", $data);
		}
    }
	   public function wlive()
    {
		#ini_set('display_errors', '1');
		
		$uid=$this->tank_auth->get_user_id();
        $this->load->view("cdn");
		$this->load->view("navheader");
		
        $rdate = $this->CModel->get_latest('racedate');
		$rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
      
		$rdrn = $this->uri->segment(3);
		$crv2 = explode("@", $rdrn)[0];
		$crn = (int)explode("@", $rdrn)[1];
		$crv = substr($crv2 ,0,2);
		#$crv = $this->CModel->get_latest('venue');cancel no auto to latest
		if(is_null($rdrn)){
		$crn = $this->CModel->get_latest('current_rn');
		$crv = $this->CModel->get_latest('venue');#only two char ST/HV/S1234
			
		}

$gr=$this->mongo_db->select(['venue','result'])->where(['racedate' => $this->mongo_db->in([$rdate,$rdate1]),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->get('hkjc_traindata');

		$data=$this->fetch_one_race($rdate,$crn,$crv,"live");#
		$data["config"] = $this->mongo_db->select(['config'])->where('uid', $uid)->getOne('hkjc_user')[0]['config'];
		if(is_null($data['crv']))
			$data=$this->fetch_one_race($rdate1,$crn,$gr[$crn-1]['venue'],"live");#
		$data["live_history"]="live";
		$data["venue_list"]=implode("|",array_column($gr,'venue'));
		$data['title']="Live | $crv | $crn";
		$data['is_admin']=$this->tank_auth->is_admin();
		$data['is_lastrn']=$crn2>=$this->CModel->get_latest('total_rn');
		
		$data['bool_samerd']= ($rdate==date('Y-m-d'));
		if($this->tank_auth->check()){
        if($data['oversea_race'])
			  $this->load->view("W_layout2", $data);
		 else
        $this->load->view("W_layout2", $data);
		}
    }
    public function live()
    {
		#ini_set('display_errors', '1');
		
		$uid=$this->tank_auth->get_user_id();
        $this->load->view("cdn");
		$this->load->view("navheader");
		
        $rdate = $this->CModel->get_latest('racedate');
		$rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
      
		$rdrn = $this->uri->segment(3);
		$crv2 = explode("@", $rdrn)[0];
		$crn = (int)explode("@", $rdrn)[1];
		$crv = substr($crv2 ,0,2);
		#$crv = $this->CModel->get_latest('venue');cancel no auto to latest
		if(is_null($rdrn)){
		$crn = $this->CModel->get_latest('current_rn');
		$crv = $this->CModel->get_latest('venue');#only two char ST/HV/S1234
			
		}
$gr=$this->mongo_db->select(['venue','result'])->where(['racedate' => $this->mongo_db->in([$rdate,$rdate1]),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->get('hkjc_traindata');

		$data=$this->fetch_one_race($rdate,$crn,$crv,"live");#
		if (!count($data['raceinfo'])){
		    	$rdate1=date('Y-m-d', strtotime("$rdate -1 day"));
		$data=$this->fetch_one_race($rdate1,$crn,$crv,"live");#
}
		$data["config"] = $this->mongo_db->select(['config'])->where('uid', $uid)->getOne('hkjc_user')[0]['config'];
		if(is_null($data['crv']))
			$data=$this->fetch_one_race($rdate1,$crn,$gr[$crn-1]['venue'],"live");#
		$data["live_history"]="live";
		$data["venue_list"]=implode("|",array_column($gr,'venue'));
		$data['title']="Live | $crv | $crn";
		$data['is_admin']=$this->tank_auth->is_admin();
		
		$data['is_lastrn']=$data["totalranrn"]>=$this->CModel->get_latest('total_rn');
		$data['bool_samerd']= ($rdate==date('Y-m-d'));
		
		
		if($this->tank_auth->check()){
         if($data['oversea_race']){
			  $this->load->view("W_layout2", $data);
			
		  }else
        $this->load->view("W_layout", $data);
		}else
		  redirect("horse");
    }

    public function history()
    {
		#ini_set('display_errors', '1'); 
$logged=($this->tank_auth->is_logged_in());
		if(!$logged)
		redirect('/auth/login/');
        $this->load->view("cdn");
		$this->load->view("navheader");
        $rdrn = $this->uri->segment(3);
        $rdate = explode("@", $rdrn)[0];
		$rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
		$crv =  explode("@", $rdrn)[1];
		$crn = intval(explode("@", $rdrn)[2]);
		
		
		$latest_rd=$this->CModel->get_latest("racedate");
		#check if venue error
		 $check_=is_null($this->mongo_db->where(['racedate' =>$rdate, 'raceno' => $crn,'venue'=>$crv])->get('hkjc_traindata')[0]['d']);
		
		if($check_){

			$crv=substr($crv,0,2);
			
		}

		
		if (empty($rdate))
			redirect();
		$data=$this->fetch_one_race($rdate,$crn,$crv,"history");
		
		
		$data["live_history"]="history";
		$gr=$this->mongo_db->select(['venue'])->where(['racedate' => $rdate,'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->get('hkjc_traindata');
		$data["venue_list"]=implode("|",array_column($gr,'venue'));
		$data['title']="History ($rdate | $crn)";
		$uid=$this->tank_auth->get_user_id();
		$data["config"] = $this->mongo_db->select(['config'])->where('uid', $uid)->getOne('hkjc_user')[0]['config'];
		$mongo_data=$this->mongo_db->where(['racedate' => $rdate, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->getOne('hkjc_traindata')[0];

		if(!empty($mongo_data['result']) and isset($mongo_data['result'])){//Ran

		 $this->load->view("V_layout", $data);
		}else if($rdate==$latest_rd and $crn>=$this->CModel->get_latest('current_rn') ){//Live not ran
		if($this->tank_auth->check())
        redirect("horse/live/$crn");
		}else
		echo "賽事被腰斬";
		
    }
	 public function switch_v(){
		 $rv=$this->input->post("rv");
		 if(!empty($rv))
		 $this->CModel->update_raceact($rv);
		 
		 
		 
	 }

    public function index()
      
    {
		$logged=($this->tank_auth->is_logged_in());
		if(!$logged)
		redirect('/auth/login/');

		#check post time to determine venue /load xml default
	
		try{
		$url =XMLJSON_Link;
$xml =web_go($url);#
$xml =$xml['INFO'];
$lastest_venue=$xml["DefaultPage"]["Venue"];
$lastest_venue2= $this->CModel->get_latest("venue");
if($lastest_venue!=$lastest_venue2)
	$lastest_venue=$lastest_venue2;

$lastest_rd=DateTime::createFromFormat("d/m/Y",$xml["DefaultPage"]["Date"])->format("Y-m-d");}catch(Throwable $e){};

		
		$rdate= (is_null($lastest_rd))?$this->input->post("rdate"):$lastest_rd;
	
		$rv=(is_null($lastest_venue))?$this->input->post("rv"):$lastest_venue;
		$rdate= (!empty($this->input->post("rdate")))?$this->input->post("rdate"):$lastest_rd;
		$rv=(!empty($this->input->post("rv")))?$this->input->post("rv"):$lastest_venue;
#print_r([$rdate,$lastest_venue2]);
       
		$glang=GOING;
		$clang=CLS2;	
		if(is_null($lastest_rd) and !isset($rdate)){
		$rdate_miss=$this->mongo_db->select(['racedate','venue','result'])->sort('inserttime', 'desc')->get('hkjc_traindata')[0];
		$rdate= (is_null($rdate2))?$rdate_miss['racedate']:$rdate2;
		$rv=(is_null($rv))?substr($rdate_miss['venue'],0,2):$rv;
		
		}
		
		$rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
#var_dump([$rdate,$rdate1]);
		if(!$logged){
		    
		   $ryear=date('Y');
		   $rmonth=date('m');
		  $paddedMonth = sprintf("%02d", $rmonth);
		$ri=$this->mongo_db->select(['racedate','raceno','venue','dist','cls','course','going','posttime','result','track'])->where('racedate',['$regex' => "$ryear-$paddedMonth"] )->where_ne('result' ,"")->sort(['inserttime'=>'desc','racedate'=> 'desc','raceno'=>'asc'])->getOne('hkjc_traindata');#one day can have ST OS

		$ri=$this->mongo_db->select(['racedate','raceno','venue','dist','cls','course','going','posttime','result','track'])->where('racedate',$ri[0]['racedate'] )->where_ne('result' ,"")->sort(['inserttime'=>'desc','racedate'=> 'desc','raceno'=>'asc'])->get('hkjc_traindata');#one day can have ST OS
   
		}else
		$ri=$this->mongo_db->select(['racedate','raceno','venue','dist','cls','course','going','posttime','result','track'])->where(['racedate' => $this->mongo_db->in([$rdate,$rdate1])])->sort('racedate', 'asc')->get('hkjc_traindata');#show same day all races
	 
		$data["raceinfo"] = $ri;
		$data['title']='即時';
		$data['not_valid_user']=(!$this->tank_auth->is_valid_user());
		$f = $this->input->post("selc");

		if (isset($f) ) {

			$d=$this->mongo_db->select(['racedate','raceno','venue','dist','cls','course','going','result','track'])->where(['racedate' =>$this->mongo_db->in([$rdate,$rdate1]),'venue'=>$this->mongo_db->in(["{$rv2}TURF", "{$rv2}AWT","{$rv}TURF", "{$rv}AWT",$rv2,$rv])])->sort('raceno', 'asc')->get('hkjc_traindata');
			
foreach($d as $r){
	$rc=$r['course'];
	if ( str_contains($r['venue'], "AWT"))
		$rc='';
else
	$rc.='欄';
	$r['result']=explode(',',$r['result']);
	$r['result']=array_slice($r['result'], 0, 4);
	$r['result']=implode(',',$r['result']);
$dd['l'][]=['rd'=>$r['racedate'],'venue'=>$r['venue'],'rn'=>$r['raceno'],'dist'=>$r['dist'],'going'=>GOING[$r['going']],'course'=>$rc,'cls'=>$clang[$r['cls']]."",'result'=>str_replace(",", " - ", $r['result'])];
}

$vv =VENUE;
$dd['wd']=get_weekday($rdate);
$dd['rv']=$vv[$d[0]['venue']];
$dd['rc']=$rc;
			echo json_encode($dd);
		}else{
			$this->load->view("cdn");
       	 $this->load->view("navheader");

		$this->load->view("menu", $data);
		}
       
    }
    public function report()
    {
#ini_set('display_errors', '1');
        $this->load->view("cdn");
      $this->load->view("navheader");


		    $rdcitr = [];
		 $rdcitr[]['$eq'] = ['$$item.isbu',1];
		 $rdcitr[]['$in'] = ['$venue',['STAWT','STTURF',"HVTURF"]];
	$r=$this->mongo_db->aggregate('hkjc_traindata', [
        ['$project' => ['_id'=>0,'racedate'=>1,'raceno'=>1,'venue'=>1,'result'=>1,'biguser'=>1,'inserttime'=>1,
 'items'=> [
            '$filter'=> [
               'input'=> '$d',
               'as'=> 'item',
               'cond'=> [ '$and'=> [...$rdcitr],
						]
						]
         ]
                
						]
     
    ],[ '$sort' => ['inserttime' => -1 ,'raceno'=>1] ]],['cursor'=>['batchSize'=>0]]);
	$data["races"] =$r;
		foreach( $data["races"] as $j=>$d){
			if(!in_array($d['venue'], ['STAWT','STTURF',"HVTURF"]))
				unset($data["races"][$j]);
			if(!array_key_exists('biguser', $d) or empty($d['biguser']) or empty($d['result'])){
				unset($data["races"][$j]);
			}
		}
		
		$data["title"] ="統計表";
		$data["races_total"] =count($data["races"]);
        $this->load->view("report", $data);
    }


   
	 public function hintssummary()
    {
     	#ini_set('display_errors', '1');
        $data["title"] = "大數據結算";
		$data["update_summary"] = 1;
        $data["hints_all"] = $this->load_all_hints();#without del

        $this->load->view("hints_admin", $data);
    }
		 public function loadrace()#combine ajax request  load chart , cctv ,wq ,cardtable
    {
	if($this->tank_auth->check()){
		  $ird= $this->input->post("crd");
			 $irn=   intval($this->input->post("crn"));
				$irv=   $this->input->post("crv");
				$crv=$irv;
					$irv = substr($irv ,0,2);
				 $isort=     $this->input->post("sort");#loadcard $curSort ,cctv $bbSort
				 $ibbsort=	  $this->input->post("bbsort");
	   $ity= $this->input->post("ty");
	     $itm = $this->input->post("tm");
		
		 $d['card']=($this->race_card($ird,$irn,$irv,$isort,$ity,$itm));
		  $d['wqq']=($this-> wqqtb($ird,$irn,$irv));
		  $d['mchart']=($this-> loadchart_2($ird,$irn,$irv));
		  $d['tv2']=$this->loadcctv2_2($ird,$irn,$irv,$ibbsort);
		    $d['tv']=$this->loadcctv_2($ird,$irn,$irv);

		echo json_encode($d);
       
		}else{
			$d=[];
			$d['msg']='EXPIRED';
				echo json_encode($d);
		  redirect("horse");
		}
		   
		 

		 
		 
		
	}
public function loadhistory2($ryear,$rmonth)
    {

		$races =$this->mongo_db->select(['venue','racedate'])->where_beth('inserttime', strtotime("$ryear-$rmonth-01 00:00:00")-28800, strtotime("last day of $ryear-$rmonth 23:59:59"))->sort('inserttime', 'desc')->get('hkjc_traindata');
		foreach($races as &$r){
			unset($r['_id']);
			if($r['venue']=='STAWT')
				$r['venue']='STTURF';
			#$r['venue']=substr($r['venue'],0,2);
		}
		
		$races= array_map("unserialize", array_unique(array_map("serialize", $races)));
		return $races[0];
	}
    public function loadhistory()
    {
		#ini_set('display_errors', '1');
	#first load no oversea
        $ryear = $this->input->post("ryear");
        $rmonth = $this->input->post("rmonth");
        $rmonth=sprintf('%02s', $rmonth);
		$rdate = $this->CModel->get_latest("racedate");
		#remove dup
		$str='';
		$paddedMonth = sprintf("%02d", $rmonth);
		$races =$this->mongo_db->select(['venue','racedate'])->where('racedate',['$regex' => "$ryear-$paddedMonth"] )->where_ne('result' ,"")->sort('inserttime', 'desc')->get('hkjc_traindata');
		#$races =$this->mongo_db->like('racedate', "$ryear-0$rmonth")->sort('inserttime', 'desc')->get('hkjc_traindata');
		#$races =$this->mongo_db->select(['venue','racedate'])->where_beth('inserttime', strtotime("$ryear-$rmonth-01 00:00:00")-28800, strtotime("last day of $ryear-$rmonth")+28800)->sort('inserttime', 'desc')->get('hkjc_traindata');#->where(['venue'=>$this->mongo_db->in(["HVTURF","STTURF","STAWT"])])
		foreach($races as &$r){
			unset($r['_id']);
			if($r['venue']=='STAWT')
				$r['venue']='STTURF';
			
		
		}
		$races= array_map("unserialize", array_unique(array_map("serialize", $races)));
$todayEnd = strtotime('today 23:59:59');
foreach ($races as $r) {
    // Skip future dates
    if (strtotime($r['racedate']) > $todayEnd) {
        continue;
    }

    $shortVenue = substr($r['venue'], 0, 2);
    $weekday    = get_weekday($r['racedate']);
    $venueName  = VENUE[$r['venue']] ?? $r['venue'];

    $str .= sprintf(
        '<a href="#" class="raceItem sRace" rel="%s|%s">' .
            '<span class="raceSelectDate" nowrap=""><div>%s %s %s</div></span>' .
            '<span class="raceClk"><i class="fa fa-chevron-right" aria-hidden="true"></i></span>' .
        '</a>',
        htmlspecialchars($r['racedate'], ENT_QUOTES),
        htmlspecialchars($shortVenue, ENT_QUOTES),
        htmlspecialchars($r['racedate']),
        htmlspecialchars($weekday),
        htmlspecialchars($venueName)
    );
}

echo json_encode("<div id='raceDateItems'>{$str}</div>");
    }
private function formatChineseAmountFull($amount)
    {
        if ($amount < 1000) return $amount;
        if ($amount >= 100000000) {
            $yi = floor($amount / 100000000);
            $remainder = $amount % 100000000;
            return $yi . '億' . ($remainder >= 1000 ? $this->formatChineseAmountFull($remainder) : '');
        }
        if ($amount >= 10000) {
            $wan = floor($amount / 10000);
            $remainder = $amount % 10000;
            return $wan . '萬' . ($remainder >= 1000 ? $this->formatChineseAmountFull($remainder) : '');
        }
        if ($amount >= 1000) {
            return floor($amount / 1000) . '千';
        }
        return $amount;
    }
	public function loadcctv_2($rdate,$crn,$crv)
    {#make html table
	 #ini_set('display_errors', '1');
	 $show_thresold=1;//over 100K 
	$short_thresold=1000;#??K
	  
		 $crv2=substr($crv,0,2);
      
		$d=$this->mongo_db->select(['wtable','ptable'])->where(['racedate' => $rdate, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->get('hkjc_cctv');
		$template = array(
       'table_open'  => '<table  id="winAllUpTB" class="alluptb" >',
        
);
    $this->table->set_template($template); 
	//sort with odd
	usort($d[0]['wtable'], function($a, $b) {
    return $a["od"] <=> $b["od"];
});
usort($d[0]['ptable'], function($a, $b) {
    return $a["od"] <=> $b["od"];
});
	foreach($d[0]['wtable'] as $r){
		
		$odd_changes="<div>{$r['od']}<i class='fas fa-long-arrow-alt-right arsize'></i>{$r['nd']}</div>";

		$jw = ["data" => $r['jw'], "class" => $r['jwcls']];
		$tw = ["data" => $r['tw'], "class" => $r['twcls']];
		
		$tmp_k=$r['amount']*1000;
	$eng_unit="{$r['amount']}K";
	$chinese_unit=$this->formatChineseAmountFull($tmp_k);
		$ma=[$r['amount'],(int)$r['amount'].'K'];
		$amt = ["data" =>"<span class={$r['amountcls']}>".$chinese_unit."</span>"];
		if($r['amount']>$show_thresold and $r['nd']>0)
	$this->table->add_row($tw,$jw,$r['hn'],$odd_changes,$amt);
	}

$a['allupW']=$this->table->generate();
$this->table->clear();
$template = array(
       'table_open'  => '<table  id="plaAllUpTB" class="alluptb" >',
      
);
    $this->table->set_template($template); 
foreach($d[0]['ptable'] as $r){
		
		$odd_changes="<div>{$r['od']}<i class='fas fa-long-arrow-alt-right arsize'></i>{$r['nd']}</div>";

		$jw = ["data" => $r['jw'], "class" => $r['jwcls']];
		$tw = ["data" => $r['tw'], "class" => $r['twcls']];
		$tmp_k=$r['amount']*1000;
	$eng_unit="{$r['amount']}K";
	$chinese_unit=$this->formatChineseAmountFull($tmp_k);
		$ma=[$r['amount'],(int)$r['amount'].'K'];
		$amt = ["data" =>"<span class={$r['amountcls']}>".$chinese_unit."</span>"];
		
		if($r['amount']>$show_thresold and $r['nd']>0)
	$this->table->add_row($tw,$jw,$r['hn'],$odd_changes,$amt);
	}
	$a['allupP']=$this->table->generate();
	return ($a);
		
	}
	public function loadcctv2_2($rdate,$crn,$crv,$sort )
    {#make html table
	# ini_set('display_errors', '1');

$crn = (int)$crn;
			
		$short_form=["W"=>"W","WIN"=>"W","P"=>"P","PLA"=>"P","Q"=>"Q","PQ"=>"QP","QP"=>"QP","D"=>"D"];
			$odd_changes="<div><i class='fas fa-long-arrow-alt-right arsize'></i></div>";
		
	
	$d=$this->CModel->get_cctv($rdate,$crn,$crv,$sort);
	
	#var_dump($rdate,$crn,$crv,$sort);
	#exit;
	foreach($d as $r){
	    $data_mins=$r["mins"];

		if($r["mins"]>1000){#today or yesterday
			$show_format='*';#yesterday
			$data_mins+=720;		
			}else{
		$show_format='';}#today
		switch($r["hlmode"]){
			
			case 1:
			$hl='Y';
			break;
			case 2:
			$hl='divG';
			break;
			case 3:
			$hl='divR';
			break;
			default:
			$hl='';
			
		}

	$r['amt']=(int)$r['amt'];
	$r['hn']=str_replace("[ ","[",$r['hn']);
	$tmp_k=$r['amt']*1000;
	$eng_unit="{$r['amt']}K";
	$chinese_unit=$this->formatChineseAmountFull($tmp_k);
	$odd_changes=$r['od']."<i class='fas fa-long-arrow-alt-right arsize'></i>".$r['nd'];
	#$odd_changes="";
	$this->table->add_row(['data' =>$show_format.$r["time"], 'rel'=>$data_mins,'class' => ""],$short_form[$r["bet_type"]],$r['hn'],$odd_changes,['data' =>"<span class='$hl'>$chinese_unit</span>", 'class' => ""]);
	}
	$a['cctv']=$this->table->generate();
	return ($a);	
	}
	public function loadcctv2()
    {#make html table
	# ini_set('display_errors', '1');

	   $rdate = $this->input->post("crd");
        $crn = (int)$this->input->post("crn");
		$crv = $this->input->post("crv");
          $sort = $this->input->post("sort");
			
		$short_form=["W"=>"W","WIN"=>"W","PLA"=>"P","Q"=>"Q","PQ"=>"QP","QP"=>"QP","D"=>"D"];
			$odd_changes="<div><i class='fas fa-long-arrow-alt-right arsize'></i></div>";
		
	
	$d=$this->CModel->get_cctv($rdate,$crn,$crv,$sort);
	foreach($d as $r){
	    $data_mins=$r["mins"];

		if($r["mins"]>1000){#today or yesterday
			$show_format='*';#yesterday
			$data_mins+=720;		
			}else{
		$show_format='';}#today
		switch($r["hlmode"]){
			
			case 1:
			$hl='Y';
			break;
			case 2:
			$hl='divG';
			break;
			case 3:
			$hl='divR';
			break;
			default:
			$hl='';
			
		}

	$r['amt']=(int)$r['amt'];
	$r['hn']=str_replace("[ ","[",$r['hn']);
	$tmp_k=$r['amt']*1000;
	$eng_unit="{$r['amt']}K";
	$chinese_unit=$this->formatChineseAmountFull($tmp_k);
	$odd_changes=$r['od']."<i class='fas fa-long-arrow-alt-right arsize'></i>".$r['nd'];
	#$odd_changes="";
	$this->table->add_row(['data' =>$show_format.$r["time"], 'rel'=>$data_mins,'class' => ""],$short_form[$r["bet_type"]],$r['hn'],$odd_changes,['data' =>"<span class='$hl'>$chinese_unit</span>", 'class' => ""]);
	}
	$a['cctv']=$this->table->generate();
	echo json_encode($a);	
	}
	public function loadcctv()
    {#make html table
	 #ini_set('display_errors', '1');
	 $show_thresold=1;//over 100K 
	$short_thresold=1000;#??K
	   $rdate = $this->input->post("crd");
	     $crv = $this->input->post("crv");
		 $crv2=substr($crv,0,2);
        $crn = (int)$this->input->post("crn");
		$d=$this->mongo_db->select(['wtable','ptable'])->where(['racedate' => $rdate, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->get('hkjc_cctv');
		$template = array(
       'table_open'  => '<table  id="winAllUpTB" class="alluptb" >',
        
);
    $this->table->set_template($template); 
	//sort with odd
	usort($d[0]['wtable'], function($a, $b) {
    return $a["od"] <=> $b["od"];
});
usort($d[0]['ptable'], function($a, $b) {
    return $a["od"] <=> $b["od"];
});
	foreach($d[0]['wtable'] as $r){
		
		$odd_changes="<div>{$r['od']}<i class='fas fa-long-arrow-alt-right arsize'></i>{$r['nd']}</div>";

		$jw = ["data" => $r['jw'], "class" => $r['jwcls']];
		$tw = ["data" => $r['tw'], "class" => $r['twcls']];
		
		$tmp_k=$r['amount']*1000;
	$eng_unit="{$r['amount']}K";
	$chinese_unit=$this->formatChineseAmountFull($tmp_k);
		$ma=[$r['amount'],(int)$r['amount'].'K'];
		$amt = ["data" =>"<span class={$r['amountcls']}>".$chinese_unit."</span>"];
		if($r['amount']>$show_thresold and $r['nd']>0)
	$this->table->add_row($tw,$jw,$r['hn'],$odd_changes,$amt);
	}

$a['allupW']=$this->table->generate();
$this->table->clear();
$template = array(
       'table_open'  => '<table  id="plaAllUpTB" class="alluptb" >',
      
);
    $this->table->set_template($template); 
foreach($d[0]['ptable'] as $r){
		
		$odd_changes="<div>{$r['od']}<i class='fas fa-long-arrow-alt-right arsize'></i>{$r['nd']}</div>";

		$jw = ["data" => $r['jw'], "class" => $r['jwcls']];
		$tw = ["data" => $r['tw'], "class" => $r['twcls']];
		$tmp_k=$r['amount']*1000;
	$eng_unit="{$r['amount']}K";
	$chinese_unit=$this->formatChineseAmountFull($tmp_k);
		$ma=[$r['amount'],(int)$r['amount'].'K'];
		$amt = ["data" =>"<span class={$r['amountcls']}>".$chinese_unit."</span>"];
		
		if($r['amount']>$show_thresold and $r['nd']>0)
	$this->table->add_row($tw,$jw,$r['hn'],$odd_changes,$amt);
	}
	$a['allupP']=$this->table->generate();
	echo json_encode($a);
		
	}
		public function loadallup2()#backend only 
    {
		$rdate = $this->CModel->get_latest('racedate');
		$crn = intval($this->CModel->get_latest('current_rn'));
		#$crv = $this->CModel->get_latest('venue');
       $w=$this->CModel->get_cctvsumwp($rdate,$crn,"W");
	   $p=$this->CModel->get_cctvsumwp($rdate,$crn,"P");
	   #var_dump($p);
	   
	}
	public function loadallup()#backend only 
    {
        #ini_set('display_errors', '1');
		#$rdate = $this->input->post("crd");
        #$crn = (int)$this->input->post("crn");
		$rdate = $this->CModel->get_latest('racedate');
		$crn = intval($this->CModel->get_latest('current_rn'));
		$crv = $this->CModel->get_latest('venue');
		 $crv2=substr($crv,0,2);
		#$rdate = "2025-04-27";
        #$crn = 9;
		if($crn<=1)
			return 1;
		$data["crd"] = $rdate;
        $data["crn"] = $crn;
		$data["cutoff_result"] = $this->mongo_db->select(['result','d'])->where(['racedate' => $rdate,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->get('hkjc_traindata');
		#may change to SQL 
		$data["wdata"]=$this->CModel->get_cctvsumwp($rdate,$crn,"W");
	  $data["pdata"]=$this->CModel->get_cctvsumwp($rdate,$crn,"P");
		/*$tjcitr[]['$eq']= ['$racedate',$rdate];
		$tjcitr[]['$eq']= ['$raceno',$crn];
		$tjcitr[]['$in']= ['$$item.type',['WIN','PLA']];
		$tjcitr[]['$lte']= ['$$item.minstogo',30];#tested 24
		$tjcitr[]['$gte']= ['$$item.minstogo',20];*/

	#print_r( $data["tdata"]);
		$mongo_data=$this->mongo_db->select(['venue','posttime','d'])->where(['racedate' => $rdate, 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne('hkjc_traindata')[0];
		$data["posttime"] =$mongo_data['posttime'];
		$data["venue"] =substr($mongo_data['venue'], 0, 2);
		$data['card']=$mongo_data['d'];
		foreach(['W','P'] as $s){

		if($crn>1)
		$a['allup'.$s]=json_encode($this->load->view("allup", $data, true));
		
		}
	print_r($a);
   
	 #is ran read html field // real live update 

    }
	 public function loadcardtable2(){
		$rdate = $this->input->post("crd");
        $crn = $this->input->post("crn");
		$mins = $this->input->post("m");
		
		 #DBLU
		$d['tdata']=$this->mongo_db->where(['racedate' => $rdate, 'raceno' => intval($crn)])->getOne('hkjc_traindata')[0];
		$d['scdata'] = $this->mongo_db->where(['racedate' => $rdate, 'raceno' => intval($crn)-1, 'minstogo' => intval($mins)])->sort('inserttime', 'desc')->getOne('hkjc_rawodd')[0]['DBLHV'];
		$d['lastdata'] = $this->mongo_db->where(['racedate' => $rdate, 'raceno' => intval($crn)-1, 'minstogo' => intval(-1)])->sort('inserttime', 'desc')->getOne('hkjc_rawodd')[0]['DBLHV'];
		
		 echo json_encode($this->load->view("cardtable2", $d,true));
	 }
	 public function load_set_punch(){
		 #7 sec for 11 races
		for ($crn = 1; $crn <= $this->CModel->get_latest("total_rn"); $crn++){
		
		 $rdate =$this->CModel->get_latest("racedate");
		 $crv =$this->CModel->get_latest("venue");
		  #$rdate = $this->input->post("crd");
		   $rdate1 = date('Y-m-d', strtotime("$rdate +1 day"));
		 $mongo_data=$this->mongo_db->where(['racedate' => $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->getOne('hkjc_traindata')[0];

		 #print_r($mongo_data);
		 	 foreach($mongo_data['d'] as $row){
		$hn=$row['hn'];
		$is_punch=($this->check_horse_lastispunch($row['name'],$rdate));
		#var_dump($hn,$is_punch);
$this->mongo_db->set(['d.'.($hn-1).'.is_lastpunch' => ($is_punch) ])->where(['racedate' => $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->update('hkjc_traindata');

	 } 
	 }
	 }
	 
	  public function race_card($crd,$crn,$crv,$sort,$ty,$tm="")
    {

	#ini_set('display_errors', '1');
		$uid=$this->tank_auth->get_user_id();
		if(!is_null($uid))
		$this->tank_auth->update_last_login_time((int)$uid);
       $rdate = $crd;
	   $data['livehistory'] = $ty;
	     $data['tm'] = $tm;
	    $rdate1 = date('Y-m-d', strtotime("$rdate +1 day"));
       
		 $crv2 = $crv;
		 $crv=substr($crv2,0,2);
		
		 $data["config"] = $this->mongo_db->select(['config'])->where('uid', $uid)->getOne('hkjc_user')[0]['config'];
		 $data["crd"] = $rdate;
		 $data["cloth_crd"] =date('Y-m-d', strtotime("$rdate -1 day"));
        $data["crn"] = $crn;
		$data["crv2"] = $crv2;
		$data["oversea_race"]=0;
		if((str_contains($crv,"S") and (!str_contains($crv,"ST"))))
		$data["oversea_race"]=1;
	#var_dump($data["oversea_race"]);
	#exit;
		$data["is_m"] =$this->agent->is_mobile();
		$data["sort_method"] =$sort;
		$data["is_valid_user"] =$this->tank_auth->is_valid_user();
        $data["current_rn"] = $this->CModel->get_latest("current_rn");
		if(!$data["oversea_race"])
		$data["hints"] =$this->load_hints_hid((int)$uid,1);
		
		$mongo_data=$this->mongo_db->where(['racedate' =>$this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->getOne('hkjc_traindata')[0];
		$data["mctime"] = $this->mongo_db->where(['racedate' => $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->sort('inserttime', 'desc')->getOne('hkjc_rawodd')[0]['inserttime'];
		
	
		$data["venue"] = $mongo_data['venue'];
		$data["biguser"] = $mongo_data['biguser'];
        $data["raceresult"] =$mongo_data['result'];
		$data["tdata"]=$mongo_data['d'];
		$data["notes_key"]="$rdate_$crv_$crn";
		#$r["abuser"]=$this->get_maxmup($mongo_data);
		$data["member"] =$this->tank_auth->get_user_profile();
		$data["hide_jtstat_display"]=($data["config"]['get_jtstat']=='0' and !is_null($data["config"]));
		if(!$data["hide_jtstat_display"])
		$data["jtstat"]=$this->loadcsv();
		$r['t']=$this->load->view("cardtable", $data, true);
		$r['range']=str_replace(",", "、", $data["biguser"]);
        return ($r);
    }
	 
	 
    public function loadcardtable()
    {

	#ini_set('display_errors', '1');
		$uid=$this->tank_auth->get_user_id();
		if(!is_null($uid))
		$this->tank_auth->update_last_login_time((int)$uid);
       $rdate = $this->input->post("crd");
	   $data['livehistory'] = $this->input->post("ty");
	     $data['tm'] = $this->input->post("tm");
	  # if(empty($rdate))
		  # $rdate = $this->CModel->get_latest("racedate");
	    $rdate1 = date('Y-m-d', strtotime("$rdate +1 day"));
        $crn = $this->input->post("crn");
		 $crv2 = $this->input->post("crv");
		 $crv=substr($crv2,0,2);
		
		 $data["config"] = $this->mongo_db->select(['config'])->where('uid', $uid)->getOne('hkjc_user')[0]['config'];
		 $data["crd"] = $rdate;
		 $data["cloth_crd"] =date('Y-m-d', strtotime("$rdate -1 day"));
        $data["crn"] = $crn;
		$data["crv2"] = $crv2;
		$data["oversea_race"]=0;
		if((str_contains($crv2,"S") and (!str_contains($crv2,"ST"))))
		$data["oversea_race"]=1;
		$data["is_m"] =$this->agent->is_mobile();
		$data["sort_method"] = $this->input->post("sort");
		$data["is_valid_user"] =$this->tank_auth->is_valid_user();
        $data["current_rn"] = $this->CModel->get_latest("current_rn");
		if(!$data["oversea_race"])
		$data["hints"] =$this->load_hints_hid((int)$uid,1);
		
		$mongo_data=$this->mongo_db->where(['racedate' =>$this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->getOne('hkjc_traindata')[0];
		$data["mctime"] = $this->mongo_db->where(['racedate' => $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->sort('inserttime', 'desc')->getOne('hkjc_rawodd')[0]['inserttime'];
		
	
		$data["venue"] = $mongo_data['venue'];
		$data["biguser"] = $mongo_data['biguser'];
        $data["raceresult"] =$mongo_data['result'];
		$data["tdata"]=$mongo_data['d'];
		$data["notes_key"]="$rdate_$crv_$crn";
		#$r["abuser"]=$this->get_maxmup($mongo_data);
		$data["member"] =$this->tank_auth->get_user_profile();
		$data["hide_jtstat_display"]=($data["config"]['get_jtstat']=='0' and !is_null($data["config"]));
		if(!$data["hide_jtstat_display"])
		$data["jtstat"]=$this->loadcsv();
		$r['t']=$this->load->view("cardtable", $data, true);
		$r['range']=str_replace(",", "、", $data["biguser"]);
        echo json_encode($r);
    }
	public function loadjtlist($rd,$typ,$rv,$list)

    {
	
		if($typ=="j")
$type="jockey";
else
$type="trainer";

		$arr[$type]=[];

		$arr[$type]=[]; 
		foreach($list as $v)
		foreach($v['d'] as $vv)
		$arr[$type][]=$vv[$type]; 
		$arr[$type] = array_unique($arr[$type]);
		return $arr[$type];
		
	
	
	}
		public function loadjtcard()
    {
		#ini_set('display_errors', '1'); 
if(!$this->tank_auth->is_valid_user()){
	echo '';
	return 1;
}
$input_p=$this->input->post("type");
$rdate = $this->input->post("crd");
$rv= $this->input->post("rv");
$rv=substr($rv,0,2);
$selected_tj=$this->input->post("tj");
$mongo_data = $this->mongo_db
    ->where(["racedate" => $rdate,'venue'=>$this->mongo_db->in(["{$rv}TURF", "{$rv}AWT",$rv])])
    ->get("hkjc_traindata");
$template = [
    "table_open" => '<table id="jttb" class="result2tb rotShow" style=width:100%>',
];
$this->table->set_template($template);
if($input_p=="j")
$th_arr = ["騎師"];
else
$th_arr = ["練馬師"];
	if(!isset($selected_tj))
$info_arr = [
'賽事班次<br>
途程<br>
賽道'.form_button('secljt','Selected',['class'=>'secljt','style'=>''])
];
else
	$info_arr = [
'賽事班次<br>
途程<br>
賽道'
];
$venue=['STTURF'=>"TURF",'HVTURF'=>"TURF","STAWT"=>'AWT'];
foreach ($mongo_data as $rn => $d) {
    $irn = $rn + 1;
    $th_arr[] = "第 $irn 場";
    #info
    $info_arr[] = ['data' => CLS2[$d["cls"]] . "<br>" . $d["dist"] . "<br>" . $venue[$d["venue"]], 'class' => 'ct'];
}
$this->table->set_heading($th_arr);
$this->table->add_row($info_arr);
#get latest list 


if($input_p=="j")
$panel=[ $this->loadjtlist($rdate,$input_p,$rv,$mongo_data),"jockey"];
else
$panel=[ $this->loadjtlist($rdate,$input_p,$rv,$mongo_data),"trainer"];

$row_tr = [];

foreach ($mongo_data as $d) {
    foreach ($panel[0] as $jt) {
        if (!in_array(trim($jt), array_column($d["d"], $panel[1]))) {
            $row_tr[$jt][] = "";
        } #no horse need to push empty string
    }
	
	
    foreach ($d["d"] as $rowd) {
		$mob=$rowd["m"];
		$yb="";
		if($rowd["rank"]<=4 and $rowd["rank"]>0)
$yb="<span class='yb'>[{$rowd["rank"]}]</span>";
	if($rowd["mob"]==1)
$mob="<span class='gb '>{$rowd["m"]}</span>";
if($rowd["mob"]==2)
$mob="<span class='rb '>{$rowd["m"]}</span>";
        #one race >2 horse race
		#$d["raceno"]=(int)$d["raceno"];
		if($rowd["w"]>0)
        $row_tr[$rowd[$panel[1]]][$d["raceno"]][] ="<div class='col'> {$rowd["name"]} $yb <br> [{$rowd["w"]}] $mob </div>";
			else
		$row_tr[$rowd[$panel[1]]][$d["raceno"]] ="<div class='col'> {$rowd["name"]} $yb <br> {$rowd["w"]} $mob </div>";
			
    }
}
#sort($row_tr);
foreach ($row_tr as $tk => $race) {
if(count(array_filter($race)) == 0)
	unset($row_tr[$tk]);
}
foreach ($row_tr as $tk => &$race) {
    foreach ($race as $rn => $hr) {
         
            if (gettype($race[$rn]) == "array") {

                $race[$rn] = implode("", $race[$rn]);
				
            }
 
        
    }
}


foreach ($row_tr as $tk => $race2) {
	
	if($tk!="-" and !empty($tk)){
		
		if(isset($selected_tj)){
			
			if(in_array($tk,$selected_tj))
    $this->table->add_row($tk, ...$race2);
		}else
	$this->table->add_row(form_checkbox('tj[]', $tk, FALSE,['id'=>"{$tk}checkbox"]).$tk, ...$race2);
	}
}

echo $this->table->generate();

    }

    public function save()
    {
        $table = $this->input->post("data");
        //$rn = $this->uri->segment(3);
        $rn = $this->input->post("rn");
       
        $this->CModel->update_tablecard(
            $table,
            $this->CModel->get_latest("racedate"),
            $rn
        );
    }

public function get_pc($val = null, $is_oversea = false) {
    if ($val === null || $val === '') {
        return 'white';
    }

    $num = floatval($val);
 if ($num ==0)
	  return 'white';
    if ($num >0 && $num < 10) {
        return 'grey';
    } elseif ($num >= 10 && $num < 20) {
        return 'green';
    } elseif ($num >= 20 && $num <= 30) {
        return 'pink';
    } else { // > 30
        return 'brown';
    }
}
    public function wqqtb($rdate,$crn ,$crv2)
    {
		#ini_set('display_errors', '1');
		$crn= intval($crn);
$crv=substr($crv2,0,2);
		$rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
	
		 $is_oversea=0;
		 if((str_contains($crv,"S") and (!str_contains($crv,"ST"))))
		$is_oversea=1;
 #var_dump(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])]);
	#exit;
      $biguser= $this->mongo_db->where(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne('hkjc_traindata')[0]['biguser'];

	   $biguser =array_map('intval', explode(',',$biguser));
	  
     
		$result = $this->mongo_db->sort('inserttime', 'desc')->where(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne('hkjc_rawodd');
		$r = $this->mongo_db->row_array($result);
		$wodd=$r['WIN'];
		$result = $this->mongo_db->sort('inserttime', 'desc')->where(['issc' => 1, 'racedate' =>  $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne('hkjc_rawodd');
		$r = $this->mongo_db->row_array($result);
		$scwodd=$r['WIN'];

		if(empty($scwodd)){
			
			$temp=$this->mongo_db->where(['racedate' => $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne('hkjc_traindata')[0]['d'];
			foreach ($temp as $t)
			$scwodd[(int)$t['hn']]=$t['scw'];

		}
$sctime = in_array($crv2, ["S1", "S2", "S3", "S4", "S5", "S6", "S7", "S8"]) ? 14 : 15;

$template = array(
    'table_open'          => '<table style="" id="pch" class="pch table-trend">',
    'heading_row_start'   => '<tr class="" style="text-align: center;">',
    'heading_row_end'     => '</tr>',
    'heading_cell_start' => '<th style="text-align: center;">',
    'heading_cell_end'   => '</th>',
    'row_start'           => '<tr class="" style="text-align: center;">',
    'row_end'             => '</tr>',
    'row_alt_start'       => '<tr class="" style="text-align: center;">',
    'row_alt_end'         => '</tr>',
    'tbody_open'          => '<tbody id="table-body">',
    'tbody_close'         => '</tbody>',
);
$this->table->set_template($template);

// 1. 初始化前置固定欄位
$th_arr = [
    '', 
    '馬號', 
    '獨前', 
    '獨贏'
];

// 2. 動態拆分：為「每分鐘」加入一個獨立的 <th>
foreach (range($sctime, -1, -1) as $w) {
	if ($w==-1)
	$th_arr[] = '<i class="fa-solid fa-siren-on" style="color:red;"></i>';
else
    $th_arr[] = '<span class="m-hdr">' . $w . '</span>';
}

// 3. 結尾警示圖示與後置欄位

$th_arr[] = '';

$this->table->set_heading($th_arr);

// 資料讀取
$db_data = $this->mongo_db->where([
    'racedate' => $this->mongo_db->in([$rdate, $rdate1]), 
    'raceno'   =>  intval($crn), 
    'venue'    => $this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT", "{$crv}AWT", $crv, $crv2])
])->getOne('hkjc_wqqp')[0]['d'] ?? [];

$temp = $this->mongo_db->where([
    'racedate' => $rdate, 
    'raceno'   => intval($crn), 
    'venue'    => $crv2
])->getOne('hkjc_traindata')[0]['d'] ?? [];

foreach ($db_data as $h => $r) {
    $tmp_hn = (string)$h;
    $key2 = "{$crv2}_{$crn}_$tmp_hn";
 $is_biguser = in_array($tmp_hn, $biguser ?? []) ? 1 : 0;
  

    // 初始化這隻馬的完整 row 陣列
    $row_cells = [];
  $cell_hn = [
    'data'         => ($is_biguser ? "<span class='biguser-block '></span>" : ''),
    'class'        => "rank-indicator",
    'data-biguser' => $is_biguser
];
    // 1. 放入前置固定欄位
    $row_cells[] = $cell_hn;
    $row_cells[] = ['data' => "<span class='horse-badge'>$tmp_hn</span>", 'class' => "qqpl$key2", 'data-name' => $tmp_hn];

    if (isset($wodd[$tmp_hn]) && $wodd[$tmp_hn] > 0) {
        $scwodd_val = (empty($scwodd[$tmp_hn])) ? "" : $scwodd[$tmp_hn];
        $row_cells[] = ['data' => $scwodd_val, 'class' => "col-odds "];
        $row_cells[] = ['data' => $wodd[$tmp_hn], 'class' => "col-odds text-amber-400 font-extrabold "];
    } else {
        $row_cells[] = ['data' => $temp[$tmp_hn - 1]['scw'] ?? '', 'class' => "col-odds "];
        $row_cells[] = ['data' => $temp[$tmp_hn - 1]['w'] ?? '', 'class' => "col-odds text-amber-400 font-extrabold "];#col-odds qqplpw$key2
    }

    // 2. 動態拆分：為「每分鐘」加入一個獨立的 <td> cell
    foreach (range($sctime, -1, -1) as $w) {
        if (isset($r[$w]) && is_array($r[$w])) {
            $u1 = $this->get_pc($r[$w]['w'] ?? null, $is_oversea);
            $u2 = $this->get_pc($r[$w]['q'] ?? null, $is_oversea);
            $u3 = $this->get_pc($r[$w]['qp'] ?? null, $is_oversea);
            $u4 = $this->get_pc($r[$w]['dbld'] ?? null, $is_oversea);

            $data_w = $r[$w]['w'] ?? 0;
            $data_q = $r[$w]['q'] ?? 0;
            $data_qp = $r[$w]['qp'] ?? 0;
            $data_dbld = $r[$w]['dbld'] ?? 0;
        } else {
            $u1 = $u2 = $u3 = $u4 = 'grey';
            $data_w = $data_q = $data_qp = $data_dbld = 0;
        }

        // 單一分鐘 cell 裡面的直立 4 小格微型區塊
        $min_cell_html = '<div style="text-align: center;" class="custom-pill-trend" data-time="' . $w . '" title="' . $w . '分鐘前">'
                        . '<div class="trend-pill ' . $u1 . '"></div>'
                        . '<div class="trend-pill ' . $u2 . '"></div>'
                        . '<div class="trend-pill ' . $u3 . '"></div>'
                        . '<div class="trend-pill ' . $u4 . '"></div>'
                        . '</div>';

        // 將此分鐘作為一個獨立 <td> 加入 row 陣列
        $row_cells[] = [
            'data'      => $min_cell_html,
            'class'     => 'trend-min-cell',
            'data-w'    => $data_w,
            'data-q'    => $data_q,
            'data-qp'   => $data_qp,
            'data-dbld' => $data_dbld
        ];
    }
	  // 3. 放入結尾警示與後置欄位
 
    $cell_hn = [
    'data'         => ($is_biguser ? "<span class='biguser-block '></span>" : ''),
    'class'        => "rank-indicator qqplpw$key2",
    'data-biguser' => $is_biguser
];
  
   # $row_cells[] = ['data' => '<i class="fa-solid fa-bolt" style="color:#eab308;"></i>', 'class' => 'status-col'];
   $row_cells[] = $cell_hn;

    // 將整個陣列傳入 add_row
	if ( $wodd[$tmp_hn] > 0) 
    $this->table->add_row($row_cells);
}

return $this->table->generate();
      
		
    }
    public function loadwqqtb()
    {
		#ini_set('display_errors', '1');
        $rdate = $this->input->post("crd");
		$rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
		 $crv2 = $this->input->post("crv");
		 $crv=substr($crv2,0,2);
		 $is_oversea=0;
		 if((str_contains($crv,"S") and (!str_contains($crv,"ST"))))
		$is_oversea=1;
        $crn = (int)$this->input->post("crn");
	
      $biguser= $this->mongo_db->where(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne('hkjc_traindata')[0]['biguser'];

	   $biguser =array_map('intval', explode(',',$biguser));
	  
       
		$result = $this->mongo_db->sort('inserttime', 'desc')->where(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne('hkjc_rawodd');
		$r = $this->mongo_db->row_array($result);
		$wodd=$r['WIN'];
		$result = $this->mongo_db->sort('inserttime', 'desc')->where(['issc' => 1, 'racedate' =>  $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne('hkjc_rawodd');
		$r = $this->mongo_db->row_array($result);
		$scwodd=$r['WIN'];

		if(empty($scwodd)){
			
			$temp=$this->mongo_db->where(['racedate' => $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne('hkjc_traindata')[0]['d'];
			foreach ($temp as $t)
			$scwodd[(int)$t['hn']]=$t['scw'];

		}
$sctime = in_array($crv2, ["S1", "S2", "S3", "S4", "S5", "S6", "S7", "S8"]) ? 14 : 15;

$template = array(
    'table_open'          => '<table style="" id="pch" class="pch table-trend">',
    'heading_row_start'   => '<tr class="" style="text-align: center;">',
    'heading_row_end'     => '</tr>',
    'heading_cell_start' => '<th style="text-align: center;">',
    'heading_cell_end'   => '</th>',
    'row_start'           => '<tr class="" style="text-align: center;">',
    'row_end'             => '</tr>',
    'row_alt_start'       => '<tr class="" style="text-align: center;">',
    'row_alt_end'         => '</tr>',
    'tbody_open'          => '<tbody id="table-body">',
    'tbody_close'         => '</tbody>',
);
$this->table->set_template($template);

// 1. 初始化前置固定欄位
$th_arr = [
    '', 
    '馬號', 
    '獨前', 
    '獨贏'
];

// 2. 動態拆分：為「每分鐘」加入一個獨立的 <th>
foreach (range($sctime, -1, -1) as $w) {
	if ($w==-1)
	$th_arr[] = '<i class="fa-solid fa-siren-on" style="color:red;"></i>';
else
    $th_arr[] = '<span class="m-hdr">' . $w . '</span>';
}

// 3. 結尾警示圖示與後置欄位

$th_arr[] = '';

$this->table->set_heading($th_arr);

// 資料讀取
$db_data = $this->mongo_db->where([
    'racedate' => $this->mongo_db->in([$rdate, $rdate1]), 
    'raceno'   => $crn, 
    'venue'    => $this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT", "{$crv}AWT", $crv, $crv2])
])->getOne('hkjc_wqqp')[0]['d'] ?? [];

$temp = $this->mongo_db->where([
    'racedate' => $rdate, 
    'raceno'   => intval($crn), 
    'venue'    => $crv2
])->getOne('hkjc_traindata')[0]['d'] ?? [];

foreach ($db_data as $h => $r) {
    $tmp_hn = (string)$h;
    $key2 = "{$crv2}_{$crn}_$tmp_hn";
 $is_biguser = in_array($tmp_hn, $biguser ?? []) ? 1 : 0;
  

    // 初始化這隻馬的完整 row 陣列
    $row_cells = [];
  $cell_hn = [
    'data'         => ($is_biguser ? "<span class='biguser-block '></span>" : ''),
    'class'        => "rank-indicator",
    'data-biguser' => $is_biguser
];
    // 1. 放入前置固定欄位
    $row_cells[] = $cell_hn;
    $row_cells[] = ['data' => "<span class='horse-badge'>$tmp_hn</span>", 'class' => "qqpl$key2", 'data-name' => $tmp_hn];

    if (isset($wodd[$tmp_hn]) && $wodd[$tmp_hn] > 0) {
        $scwodd_val = (empty($scwodd[$tmp_hn])) ? "" : $scwodd[$tmp_hn];
        $row_cells[] = ['data' => $scwodd_val, 'class' => "col-odds "];
        $row_cells[] = ['data' => $wodd[$tmp_hn], 'class' => "col-odds text-amber-400 font-extrabold "];
    } else {
        $row_cells[] = ['data' => $temp[$tmp_hn - 1]['scw'] ?? '', 'class' => "col-odds "];
        $row_cells[] = ['data' => $temp[$tmp_hn - 1]['w'] ?? '', 'class' => "col-odds text-amber-400 font-extrabold "];#col-odds qqplpw$key2
    }

    // 2. 動態拆分：為「每分鐘」加入一個獨立的 <td> cell
    foreach (range($sctime, -1, -1) as $w) {
        if (isset($r[$w]) && is_array($r[$w])) {
            $u1 = $this->get_pc($r[$w]['w'] ?? null, $is_oversea);
            $u2 = $this->get_pc($r[$w]['q'] ?? null, $is_oversea);
            $u3 = $this->get_pc($r[$w]['qp'] ?? null, $is_oversea);
            $u4 = $this->get_pc($r[$w]['dbld'] ?? null, $is_oversea);

            $data_w = $r[$w]['w'] ?? 0;
            $data_q = $r[$w]['q'] ?? 0;
            $data_qp = $r[$w]['qp'] ?? 0;
            $data_dbld = $r[$w]['dbld'] ?? 0;
        } else {
            $u1 = $u2 = $u3 = $u4 = 'grey';
            $data_w = $data_q = $data_qp = $data_dbld = 0;
        }

        // 單一分鐘 cell 裡面的直立 4 小格微型區塊
        $min_cell_html = '<div style="text-align: center;" class="custom-pill-trend" data-time="' . $w . '" title="' . $w . '分鐘前">'
                        . '<div class="trend-pill ' . $u1 . '"></div>'
                        . '<div class="trend-pill ' . $u2 . '"></div>'
                        . '<div class="trend-pill ' . $u3 . '"></div>'
                        . '<div class="trend-pill ' . $u4 . '"></div>'
                        . '</div>';

        // 將此分鐘作為一個獨立 <td> 加入 row 陣列
        $row_cells[] = [
            'data'      => $min_cell_html,
            'class'     => 'trend-min-cell',
            'data-w'    => $data_w,
            'data-q'    => $data_q,
            'data-qp'   => $data_qp,
            'data-dbld' => $data_dbld
        ];
    }
	  // 3. 放入結尾警示與後置欄位
 
    $cell_hn = [
    'data'         => ($is_biguser ? "<span class='biguser-block '></span>" : ''),
    'class'        => "rank-indicator qqplpw$key2",
    'data-biguser' => $is_biguser
];
  
   # $row_cells[] = ['data' => '<i class="fa-solid fa-bolt" style="color:#eab308;"></i>', 'class' => 'status-col'];
   $row_cells[] = $cell_hn;

    // 將整個陣列傳入 add_row
	if ( $wodd[$tmp_hn] > 0) 
    $this->table->add_row($row_cells);
}

echo $this->table->generate();
      
		
    }
    public function loadchart_2($rdate, $crn,$crv2)
   
    {#ini_set('display_errors', '1');
	
      
		$rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
  
		 $crv=substr($crv2,0,2);
		#if(in_array($crv,['ST',"HV"])){
        $chartdata = $this->mongo_db->where(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->get('hkjc_chart')[0]['d'];
		
		$r['chart']=$chartdata;
		#}
	if($chartdata){
		list($r['tl'],$r['ztime'])=$this->loadmft([$rdate],$crn,$crv);
		$r['d2']=$this->loadmft_live([$rdate,$rdate1],$crn,$crv);
	}
		return $r;
        
    }
	
    public function loadchart()
   
    {#ini_set('display_errors', '1');
	
        $rdate = $this->input->post("crd");
		$rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
        $crn = $this->input->post("crn");
		 $crv2 = $this->input->post("crv");
		 $crv=substr($crv2,0,2);
		#if(in_array($crv,['ST',"HV"])){
        $chartdata = $this->mongo_db->where(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->get('hkjc_chart')[0]['d'];
		
		$r['chart']=$chartdata;
		#}
	if($chartdata){
		list($r['tl'],$r['ztime'])=$this->loadmft([$rdate],$crn,$crv);
		$r['d2']=$this->loadmft_live([$rdate,$rdate1],$crn,$crv);
	}
		echo json_encode($r,true);
        
    }

    public function quickinfo()
    {
	
         $rdate = $this->input->post("crd");
		 $rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
        $crn = $this->input->post("crn");
		 $crv2 = $this->input->post("crv");
		 $crv=substr($crv2,0,2);
       $ri=$this->mongo_db->select(['scr','dist','cls','course','going','posttime','venue','track'])->where(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1]),'raceno'=>intval($crn),'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->sort('raceno', 'asc')->getOne('hkjc_traindata')[0];
	  $rc=$ri['course'];
	  $g=GOING[$ri['going']];
	if ($ri['course']=='ALL WEATHER TRACK'){
		$rc='泥地';
		$g='';
	}
 $ri['posttime']=explode(' ',$ri['posttime'])[1];
$ri['going']=$g;
$ri['cls']=CLS2[$ri['cls']];
$ri['curr_v']=$ri['venue'];
$ri['curr_track']=$ri['track'];
 $ri['venue']=substr($ri['venue'],0,2);
$ri['course']=$rc;

		$mongo_data=$this->mongo_db->where(['racedate' =>$this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn),'venue'=>$this->mongo_db->in(["{$crv}TURF", "{$crv}AWT",$crv])])->getOne('hkjc_traindata')[0];
	
		$runner_count=count($mongo_data['d']);
		$s=[];
		foreach(range(1,$runner_count) as $hi){
			if($hi==$ri['scr'])
				$s[]=0;
			else
				$s[]=1;
			
		}
		
 $ri['scr']=join('|',$s);
 
		unset($ri['_id']);
	 
		 $_beforerdate=strtotime($rdate.'-5 day');
		 $_nextrdate=strtotime($rdate.'+5 day');
		$r=$this->mongo_db->select(['racedate','venue'])->where_beth('inserttime', $_beforerdate-1, $_nextrdate+1)->get('hkjc_traindata');
		$date_list=[];
		foreach($r as $a) {
		$date_list[]=$a['racedate'];
		$venue_list[]=$a['venue'];
		
		}
		$result = array_unique($date_list);
		$result=array_diff( $result, [$rdate] ) ;
		$ri['b']=$result[0];
		$ri['bv']=$r[0]['venue'];#substr($r[0]['venue'],0,2);
		if(empty($result[0])){
			$ri['b']=$this->loadhistory2(explode('-',$rdate)[0],7)['racedate'];
			
			
		}
		 $num_row=count($this->mongo_db->where(['racedate'=> end($result),'venue'=>end($venue_list)])->get('hkjc_traindata'));
		 if($num_row>0){
		$ri['n']=end($result);#date
		
		$ri['nv']=end($venue_list);#substr(end($venue_list),0,2);
		if($ri['n']==$ri['b'])
			$ri['n']=$this->loadhistory2(explode('-',$rdate)[0],9)['racedate'];
		 }
        echo json_encode($ri);
    }
function push_msg(){
	 $rd=$this->CModel->get_latest('racedate');
	$rn=(int)$this->CModel->get_latest('current_rn');
	 $message = $this->input->post("txt");
	$this->mongo_db->push('msg', ['itime'=>strtotime('now'),'txt'=>$message])->where(['racedate' => $rd, 'raceno' => $rn ])->update('hkjc_traindata');
	

}
function del_msg(){
	 $rd=$this->CModel->get_latest('racedate');
	$rn=(int)$this->CModel->get_latest('current_rn');
	
	$this->mongo_db->set('msg', [])->where(['racedate' => $rd, 'raceno' => $rn ])->update('hkjc_traindata');
	

}
function load_msg($a,$b){
	 $ri=[];
		 $msg=$this->mongo_db->select(['msg'])->where(['racedate' =>$a,'raceno' => (int)$b])->getOne('hkjc_traindata')[0]['msg'];
	
	foreach($msg as $a){
			$mt=date('H:i',$a['itime']);
		$ri[]="[$mt] {$a['msgt']}";
		
	}
	return $ri;

}
function broadcast(){
	#ini_set('display_errors', '1');
	$rd=$this->CModel->get_latest('racedate');
	$rn=(int)$this->CModel->get_latest('current_rn');
		$this->load->view("cdn");
		$d['title']='訊息平台';
		$d['mdata']=$this->load_msg($rd,$rn);
		$d['card']=$this->mongo_db->where(['racedate' => $rd, 'raceno' => $rn])->getOne('hkjc_traindata')[0]['d'];
		$this->load->view("msg_panel",$d);
	
	
	}
public function misc_display()
    {#ini_set('display_errors', '1');
		  $rdate = $this->input->post("crd");
		   $rdate1=date('Y-m-d', strtotime("$rdate +1 day"));
        $crn = intval($this->input->post("crn"));
		 $crv2 = $this->input->post("crv");
		 $crv=substr($crv2,0,2);
		#$crv=$this->CModel->get_latest('venue');

		if(!isset($crn))
			return 1;
$result = $this->mongo_db->sort('inserttime', 'desc')->select(['pooltot'])->where(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1]), 'raceno' => intval($crn)	,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne('hkjc_rawodd');
		
$r = $this->mongo_db->row_array($result);
	unset($r['_id']);
	$ri['p']=['WIN'=>'','PLA'=>'','QIN'=>'','QPL'=>'','DBL'=>''];
	if(!empty($r['pooltot']))
	$ri['p']=$r['pooltot'];

	 $dt=$this->mongo_db->select(['venue','posttime'])->where(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1])	,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->get('hkjc_traindata');
	 foreach ($dt as $vn=>$v){
		$v=$v['posttime'];
		 $ri['vl']=($vn+1);
$start = new DateTime($v);

$end = new DateTime(date("Y-m-d H:i"));

$d=$end->diff($start); 
$day=$d->format('%r%d');
$h=$d->format('%r%H');
$m=$d->format('%r%i');

$remind_mins=(1440*$day+60*$h+$m)-1;
$remind_mins=($remind_mins>0)?$remind_mins:0;
$ri['d'][]=$remind_mins;


	 }
	 	$utime = $this->mongo_db->sort("inserttime", "desc")->select(['inserttime'])->where(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1])	,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne("hkjc_rawodd")[0];
	 $ri['unixtime']=$utime['inserttime'];
	 #Check show time 
	
	 $msg=$this->mongo_db->select(['msg'])->where(['racedate' =>  $this->mongo_db->in([$rdate,$rdate1]),'raceno' => $crn,'venue'=>$this->mongo_db->in(["{$crv2}TURF", "{$crv}TURF", "{$crv2}AWT","{$crv}AWT",$crv,$crv2])])->getOne('hkjc_traindata')[0]['msg'];
	 if(!is_null($msg)){
	 $ri['msgcount']=count($msg);
	 if(!$this->tank_auth->is_testuser())
	foreach(array_reverse($msg) as $id=>$a){
			$mt=$a['t'];
		$ri['msgs'][]="$id@@$mt@@{$a['msgt']}";

	}if(count($msg)>0){
	$ri['msg']=implode("####",$ri['msgs']);
	unset($ri['msgs']);}
	}

		  echo json_encode($ri);
	}

public function rsupdate()
    {
		#ini_set('display_errors', '1');
		$data["venue"] =  $this->CModel->get_latest("venue");
		$data["lu"] =  $this->CModel->get_latest("racedate");
		$this->load->view("rsupdate", $data);
    }
	


	  public function myhints()
    {
		
		
		$data["admin"] =$this->tank_auth->is_admin();
		$x=$this->tank_auth->get_user_profile();
		
		$data["hints_all"] =$this->load_hints_hid((int)$x->id,0);#load_hints((int)$x->id,"");
	
       $this->load->view("hints",$data);
    }

	  public function load_hints_hid($uid,$on_only=0)
    {
		if($uid>0){#user logined
			$hid_arr=$this->mongo_db->where(['uid' =>$uid])->getOne('hkjc_user')[0]['hints'];#user hid list 
			$arr=array_column($hid_arr,'hid');#can empty
		if(empty($arr))
		$arr=[0];
	}	else
		
			$arr=[0];
	
			$h_arr=$this->mongo_db->where(['hid' => $this->mongo_db->in($arr)])->get('hkjc_hints');#should be not display_XXX
		foreach($h_arr as $ri=>$rv){
		$h_arr[$ri]['param']['hid']=$h_arr[$ri]['hid'];
		$i=array_search($h_arr[$ri]['hid'],$arr);
		$h_arr[$ri]['param']['display_text']=trim($hid_arr[$i]['display_text']);#is each user config 
		$h_arr[$ri]['param']['display_style']=$hid_arr[$i]['display_style'];
		$h_arr[$ri]['param']['display_onoff']=$hid_arr[$i]['display_onoff'];
	    $h_arr[$ri]['param']['del_at']=$hid_arr[$i]['del_at'];
				if($h_arr[$ri]['param']['display_onoff']=="1")
					$on_arr[]=$h_arr[$ri]['param'];
				
		$h_arr2[]=$h_arr[$ri]['param'];
		}
		if($on_only)
				return $on_arr;
		return $h_arr2;
		
	}
	
	
	  public function load_all_hints()
    {#hints summary
		
	
		$h_arr2=[];
	
			$h_arr=$this->mongo_db->get('hkjc_hints');

		foreach($h_arr as $ri=>$rv){
		$h_arr[$ri]['param']['hid']=$h_arr[$ri]['hid'];
		$h_arr2[]=$h_arr[$ri]['param'];
		}
		
		return $h_arr2;
    }
	  public function load_allhints()#user display setting
    {
		
	
		$h_arr2=[];
		$h_arr=$this->mongo_db->where_gte('uid' ,3)->get('hkjc_hints');
		
		foreach($h_arr as $ri=>$rv){
			$uname=$this->tank_auth->get_user_profile_by_id($rv['uid'])->username;
		$h_arr[$ri]['param']['hid']=$h_arr[$ri]['hid'];
			$hid_arr=$this->mongo_db->where(['uid' =>(int)$rv['uid']])->getOne('hkjc_user')[0]['hints'];#user hid list 
			$arr=array_column($hid_arr,'hid');
		$i=array_search($h_arr[$ri]['hid'],$arr);
		$h_arr[$ri]['param']['display_text']=trim($hid_arr[$i]['display_text']);#is each user config 
		$h_arr[$ri]['param']['display_style']=$hid_arr[$i]['display_style'];
		$h_arr[$ri]['param']['display_onoff']=$hid_arr[$i]['display_onoff'];
        $h_arr[$ri]['param']['del_at']=$hid_arr[$i]['del_at'];
		
		
		$h_arr[$ri]['param']['uid']=$uname;
		$h_arr2[]=$h_arr[$ri]['param'];
		}
		
		return $h_arr2;
    }

	  public function userhints()
    {
		
	
		if($this->tank_auth->is_admin()){

	$data =$this->mongo_db->where_gte('uid' ,3)->get('hkjc_hints');

			
		$r2=$this->load_allhints();


	
		$data["hints_all"]=$r2;
			
       $this->load->view("hints",$data);
	   
		}
    }
    public function savehints()
    {
       
        $d = $this->input->raw_input_stream;
        parse_str($d, $arr);
		
        $this->CModel->set_hints($arr, "save");
    }
    public function updatehints()
    {
       
        $d = $this->input->raw_input_stream;
        parse_str($d, $arr);
        $this->CModel->set_hints($arr, "update");
    }
    public function deletehints()
    {
        
        $d = $this->input->raw_input_stream;
        parse_str($d, $arr);
        $this->CModel->set_hints($arr, "delete");
    }
	 public function quickhints()
    {
		 $this->queryhints();
	}
	 public function queryhints($search_param='')
    {
   ini_set('display_errors', '1');
	#total can zero dropdown menu remove 
	
    $id = $this->input->post('d');
	
	$bet=$id['bet'];
	$data['bet']=$bet;
    $tjcitr = [];
    $sbcitr = [];
    $rdcitr = [];
    $lang = [
        "w_p" => "w%",
        "q_p" => "q%",
        "qpl_p" => "qpl%",
        "dblu_p" => "dblu%",
        "dbld_p" => "dbld%",
		"w_p2" => "w%2",
        "q_p2" => "q%2",
        "qpl_p2" => "qpl%2",
        "dblu_p2" => "dblu%2",
        "dbld_p2" => "dbld%2",
		

    ];
	$only_hn=0;
	foreach($id as $l=>$value)
	if(!isset($value))
		unset($id[$l]);
		$rdcitr[]['$ne'] = ['$$item.draw',0];
		$rdcitr[]['$ne'] = ['$$item.draw','-'];
		 $rdcitr[]['$ne'] = ['$$item.draw','--'];
		 $sbcitr[]['$ne']=['$$item.w',0];
		 $sbcitr[]['$ne']=['$$item.m3',0];
		  $sbcitr[]['$ne']=['$$item.m','-'];
		  $tjcitr[]['$ne'] = ['$$item.jockey', '-'];
		   $tjcitr[]['$ne'] = ['$$item.jockey', '--'];
		   $tjcitr[]['$ne'] = ['$$item.scw', 0];
		   $tjcitr[]['$gte'] = ['$inserttime',0];
		   	$tjcitr[]['$in'] = ['$venue',['STTURF','HVTURF','STAWT']];
	  	$tjcitr[]['$ne'] = ['$course',""];
	foreach ($id as $k => $v) {
		
		switch($k){
			case "name":
		$only_hn=1;
			
				$search_hname =$v;
$str_length   = mb_strlen($search_hname);

$rdcitr[] = [
    '$or' => [
        [ '$eq' => [ [ '$substrCP' => ['$$item.name', 0, $str_length] ], $search_hname ] ],
        [ '$eq' => [ [ '$substrCP' => ['$$item.name', 1, $str_length] ], $search_hname ] ],
        [ '$eq' => [ [ '$substrCP' => ['$$item.name', 2, $str_length] ], $search_hname ] ],
        [ '$eq' => [ [ '$substrCP' => ['$$item.name', 3, $str_length] ], $search_hname ] ],
        [ '$eq' => [ [ '$substrCP' => ['$$item.name', 4, $str_length] ], $search_hname ] ]
    ]
];
			break;
			case "rank":
			case "draw": 
			 if (!empty($v)) {
            $is1 = array_map("floatval",  $v);
			$is2 = array_map("strval",  $v);#may be string
            $rdcitr[]['$in'] = ['$$item.' . $k, array_merge($is1 , $is2)];
		
        }
		 $tjcitr[]['$ne'] = ['$$item.' . $k,''];
		
			break;
		case "trainer":
		case "jockey": 
		
		 if (isset($v) and !empty($v)) {
			
			$tjcitr[]['$in'] = ['$$item.' . $k,explode(',',join(',',$v))];
           
			
        }
		break;
		case "season":
			if($v==0){
				$min=2017;
				$max=date('Y')+1;
			}else
			list($min,$max)=explode('-',$v);
			$tjcitr[]['$gte'] = ['$inserttime',strtotime("$min-09-01")];
			$tjcitr[]['$lte'] = ['$inserttime',strtotime("$max-07-31")];
		break;
		case "track":
		
		$tjcitr[]['$in'] = ['$venue',explode(',',join(',',$v))];
		break;
		case "dist":
		
		$tjcitr[]['$in'] = ['$dist',explode(',',join(',',$v))];
		break;
		case "course":
		
		$tjcitr[]['$in'] = ['$course',explode(',',join(',',$v))];
		break;
		case "bet":
		break;
		case "isbu":
		if((int)$v<2)
		 $sbcitr[]['$eq']=['$$item.isbu',(int)$v];
		break;

		
		default:
		  $v = floatval($v);
        $coverted_key=(in_array($k, array_keys($lang)))?$lang[$k]:$k;
            if (str_contains($coverted_key,'2')) {//<=
			 $coverted_key = rtrim($coverted_key, "2");
				
                $sbcitr[]['$lte'] = ['$$item.' .  $coverted_key, floatval($v)];
					#$sbcitr[]['$eq']=['$$item.'. $coverted_key,''];
            } else{//>=
				
				  $coverted_key = rtrim($coverted_key, "2");
                $sbcitr[]['$gte'] = ['$$item.' .  $coverted_key, floatval($v)];
				$sbcitr[]['$ne']=['$$item.'. $coverted_key,''];
				 
             
            }
		
        
    }}
	

if($only_hn)
	$query_arr=[...$rdcitr];
else
	$query_arr=[...$tjcitr,...$sbcitr,...$rdcitr];

#echo '<pre>', print_r($query_arr, true), '</pre>';
$r=$this->mongo_db->aggregate('hkjc_traindata', [
        ['$project' => ['_id'=>0,'racedate'=>1,'raceno'=>1,'venue'=>1,'inserttime'=>1,
 'items'=> [
            '$filter'=> [
               'input'=> '$d',
               'as'=> 'item',
               'cond'=> [ '$and'=>$query_arr,
						]
						]
         ]
                
						]
        
    ],['$unwind'=> '$items'],[ '$sort' => ['inserttime' => -1 ] ]],['cursor'=>['batchSize'=>1000]]);


$data['r']=$r;
if ((count($r)>0 and $this->race_is_pending()) or $this->tank_auth->is_admin() )
echo  $this->load->view("bigdata_query",$data,true);
else echo ''; 
    }
public function race_is_pending()
{
	$rdate = $this->CModel->get_latest('racedate');
		$whole=$this->mongo_db->select(['posttime'])->where(['racedate' => $rdate])->get('hkjc_traindata');
		
		$tdate=strtotime($whole[0]['posttime']);
		$tdate2=strtotime($whole[count($whole)-1]['posttime']);
		
		$cdate=strtotime("+30 minutes");
		$cdate2=strtotime("now");
	return (($cdate<=$tdate) or ($cdate2>=$tdate2));
}
    public function bigdata()
    {if(!$this->tank_auth->is_logged_in())
		return;
		$logged=($this->tank_auth->is_logged_in());
		if(!$logged)
		redirect('/auth/login/');
	
	
	  $csrf = array(
        'name' => $this->security->get_csrf_token_name(),
        'hash' => $this->security->get_csrf_hash()
);
		$data["title"] = '分析數據';
		$rdate = $this->CModel->get_latest('racedate');
		$whole=$this->mongo_db->select(['posttime'])->where(['racedate' => $rdate])->get('hkjc_traindata');
		
		$tdate=strtotime($whole[0]['posttime']);
		$tdate2=strtotime($whole[count($whole)-1]['posttime']);
		
		$cdate=strtotime("+30 minutes");
		$cdate2=strtotime("now");
		$data["csrf"] =$csrf;
		$data["admin"] =$this->tank_auth->is_admin();
		$data["race_is_pending_b30"] = $this->race_is_pending();
		
        $this->load->view("cdn");
		$this->load->view("navheader");
		
        $this->load->view("bigdata",$data);
    }
