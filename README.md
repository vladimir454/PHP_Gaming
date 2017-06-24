<pre>

<?php

namespace Admin\Controller;
use Think\Controller;

class MemberController extends BaseController{
	
	public function index(){
		$status = intval(I('status'));
		$search_user = I('search_user');
		$content = I('content');
		$start = I('startime');
		$end = I('endtime');

		$data['status'] = $status;
		$data['search_user'] = $search_user;
		$data['content'] = $content;

		$where = array();

		if (!empty($search_user) and $content) {
			switch ($search_user) {
				case '1':
					# code..
					$condition['id'] = $content;
					$condition['vip'] = 1;
					break;
				case '2':
					$condition['nickname'] = $content;
					$condition['vip'] = 0;
					break;
				case '3':
					$condition['user_agent'] = $content;
					break;
				case '4':
					$content['nickname'] = $content;
					$condition['vip'] = 0;
					break;
				case '5':
					$condition['id'] = $content;
					$condition['vip'] = 0;
			}
		} else {
			$condition['id'] = $content;
			$condition['user_agent'] = $content;
			$condition['nickname'] = array('like', "%{$content}%");
			$condition['_logic'] = 'OR';
		}


		if (!empty($status))
			$map['status'] = $status;

		if (!empty($start)) {
			$map['reg_time'] = array();
		}

		if (!empty($end)) {
			$map['reg_time'] = array();
		}

		$where['_complex'] = array($map, $condition);
		$where['_logic'] = 'AND';

        if($nickname){
            $member = M('user');
        	$count = $member->where($where)->count();
            $page = new \Think\Page($count,10);
            $show = $page->show();
            $list = $member->where($where)->limit($page->firstRow.','.$page->listRows)->order("points DESC")->select();
        }else{
            $member = M('user');
            $count = $member->count();
            $page = new \Think\Page($count,10);
            $show = $page->show();
            $list = $member->where($where)->limit($page->firstRow.','.$page->listRows)->order("points DESC")->select();
        } 
		foreach($list as $key => $value){
            $list[$key]['add_point'] = D('integral')->where('uid = %d and type = %d',$value['id'], 1)->sum('points');
            $list[$key]['del_point'] = D('integral')->where('uid = %d and type = %d',$value['id'], 0)->sum('points');
            $list[$key]['income'] = D('order')->where('userid = %d',$value['id'])->sum('add_points');
            $list[$key]['expense'] = D('order')->where('userid = %d',$value['id'])->sum('del_points');
            $list[$key]['user_agent'] = unserialize($value['user_agent'])['device'].'/'.unserialize($value['user_agent'])['name'].'('.unserialize($value['user_agent'])['version'].')';
        }


        
		$this->assign('show',$show);
		$this->assign('condition', $data);
		$this->assign('list',$list);
		$this->display();
	}
    
	public function del(){
		$id = I('id');
		M('user')->where("id = $id")->delete();
		M('wx')->where("userid = $id")->delete();
		M('order')->where("userid = $id")->delete();
		M('message')->where("uid = $id")->delete();
		M('integral')->where("uid = $id")->delete();
		M('quest')->where("uid = $id ")->delete();
		$this->success("删除成功");
	}
	
	
	public function disable(){
		$id = I('id');
		$res = M('user')->where("id = $id")->setField('status',0);
		if($res){
			$this->success('禁用成功！');
		}else{
			$this->error('禁用失败！');
		}
	}
	
	
	public function endisable(){
		$id = I('id');
		$res = M('user')->where("id = $id")->setField('status',1);
		if($res){
			$this->success('启用成功！');
		}else{
			$this->error('启用失败！');
		}
	}

	//设置推荐
	public function setrec(){
		$db = M('recset');
		$rs = $db->limit('1')->select();
		$list = $db->select();
		$this->assign('rs1',$rs[0]);
		$this->assign('list',$list);
		$userid = I('id');
		$this->assign('userid',$userid);
		$db = M('user');
		$list = $db->where("id=$userid")->field('id,nickname,agent_level,bk_mode,reced_id')->select();
		$this->assign('list2',$list[0]);
		$this->display();
	}

	//设置推荐 - 推荐人ajax
	public function actrec(){
		$db = M('user');
		$userid = I('userid');
		$data['agent_level'] = I('rec_level');
		$data['bk_mode'] = I('rec_bkmoney');
		$data['reced_id'] = I('recid');
		$rs = $db->where("id=$userid")->save($data);
		if($rs){
			$this->success('操作成功！');;
		}else{
			$this->success('操作成功！');;
		}
		//$this->display();
		
	}

	public function bkmoney(){
		$db = M('recset');
		$rs = $db->limit('1')->select();
		$list = $db->select();
		$this->assign('rs1',$rs[0]);
		$this->assign('list',$list);
		$this->display();
	}


	public function backset(){
		$db = M('recset');
		$mrrate = I('m_return_ratio');// 用户反水比例
		$agent = I('agent');//代理模式 1 关闭 2 开启
		$mode = I('mode'); //提成模式 1 按下注金额  2按照平台盈利
		$mcount = I('mcount');//结算模式 1 自动  2 人工结算 自动结算在凌晨0:00开始结算前一个自然日数据。
		$userbk = I('userbk');//用户反水设置： 1 关闭 2 开启
		$data['mrrate'] = $mrrate;
		$data['agent'] = $agent;
		$data['mode'] = $mode;
		$data['mcount'] = $mcount;
		$data['userbk'] = $userbk;


		$xiarate1 = I('xiarate1');
		$pingrate1 = I('pingrate1');
		$userrate1 = I('userrate1');
		$data['xiarate'] = $xiarate1;
		$data['pingrate'] = $pingrate1;
		$data['userrate'] = $userrate1;
		$db->where('id=1')->save($data);

		$xiarate2 = I('xiarate2');
		$pingrate2 = I('pingrate2');
		$userrate2 = I('userrate2');
		$data['xiarate'] = $xiarate2;
		$data['pingrate'] = $pingrate2;
		$data['userrate'] = $userrate2;
		$db->where('id=2')->save($data);

		$xiarate3 = I('xiarate3');
		$pingrate3 = I('pingrate3');
		$userrate3 = I('userrate3');
		$data['xiarate'] = $xiarate3;
		$data['pingrate'] = $pingrate3;
		$data['userrate'] = $userrate3;
		$db->where('id=3')->save($data);

		$xiarate4 = I('xiarate4');
		$pingrate4 = I('pingrate4');
		$userrate4 = I('userrate4');
		$data['xiarate'] = $xiarate4;
		$data['pingrate'] = $pingrate4;
		$data['userrate'] = $userrate4;
		$db->where('id=4')->save($data);

		$xiarate5 = I('xiarate5');
		$pingrate5 = I('pingrate5');
		$userrate5 = I('userrate5');
		$data['xiarate'] = $xiarate5;
		$data['pingrate'] = $pingrate5;
		$data['userrate'] = $userrate5;
		$db->where('id=5')->save($data);

		$xiarate6 = I('xiarate6');
		$pingrate6 = I('pingrate6');
		$userrate6 = I('userrate6');
		$data['xiarate'] = $xiarate6;
		$data['pingrate'] = $pingrate6;
		$data['userrate'] = $userrate6;
		$db->where('id=6')->save($data);


		$xiarate7 = I('xiarate7');
		$pingrate7 = I('pingrate7');
		$userrate7 = I('userrate7');
		$data['xiarate'] = $xiarate7;
		$data['pingrate'] = $pingrate7;
		$data['userrate'] = $userrate7;
		$db->where('id=7')->save($data);

		$xiarate8 = I('xiarate8');
		$pingrate8 = I('pingrate8');
		$userrate8 = I('userrate8');	
		$data['xiarate'] = $xiarate8;
		$data['pingrate'] = $pingrate8;
		$data['userrate'] = $userrate8;
		$db->where('id=8')->save($data);
		$this->success('操作成功！');
	}

	public function basic(){
		if(IS_POST){
			$admin = session('admin');
			$admin_id = $admin['id'];
			echo $admin_id;die;
			$data['newmember'] = I('newmember');
			$data['maitain'] = I('maitain');
			$data['main_desc'] = I('main_desc');
			$data['welcom'] = I('welcom');
			$data['chat'] = I('chat');
			$data['roboter'] = I('roboter');
			$data['random'] = I('random');
			$data['badwords'] = I('badwords');
			$data['up'] = I('up');
			$data['up_error'] = I('up_error');
			$data['down'] = I('down');
			$data['down_cancel'] = I('down_cancel');
			$data['customer_welcom'] = I('customer_welcom');
			print_r($data);die;
			//$this->success('操作成功！');
			exit();
		}
		$this->display();
	}

	public function vip_list(){ // 会员列表-------------------------


		$this->display('share');

	}

	public function manage(){  //········会员类型管理
		$tb = D('user');
		if ($tb->create()) {
			$result = $tb->add();
		
			if ($result) {
				$this->success('add');
			}else{
				$this->error('wrong');
			}

		}

		$this->display();
	}

	//..............会员统计
	//@author : @hotmail.com
	public function statics(){ 
		//actualy week
		$db = M('user');
		
		$test = $db->find();

		$start = date("Y-m-d H:i:s",mktime(0,0,0,date("m"),date("d")-(date("w")+6)%7,date("Y")));
		$end = date("Y-m-d H:i:s",mktime(23,59,59,date("m"),date("d")-(date("w")+6)%7+6,date("Y")));

		$change = strtotime($start);
		$echange= strtotime($end);

		$res_normal = $db->field("count('*') as sum")
		->where("vip=0 and reg_time >= $change and reg_time <= $echange")->select();

		$res_vip = $db->field("count('*') as sum")
		->where("vip=1 and reg_time >= $change and reg_time <= $echange")->select();	

		// var_dump($change,$echange);
 		// var_dump($res_normal, $res_vip);exit;

		//actualy month
		$begin_month = mktime(0,0,0, date('m'), 1, date('Y'));
		$end_month = mktime(23,59,59,date('m'),date('t'),date('Y'));

		$begin_month_res= date('Y-m-d',$begin_month); //change month to human time
		$end_month_res = date('Y-m-d',$end_month);// change month to human time
		// var_dump($begin_month_res, $end_month_res);exit();

		$res_month = $db->field("count('*') as sum")
		->where("vip=0 and reg_time >= $begin_month and reg_time <= $end_month")->select();

		$res_month2 = $db->field("count('*') as sum")
		->where("vip=1 and reg_time >= $begin_month and reg_time <= $end_month")->select();	

		$all_members = $db->field("count('*') as sum")
		->where("vip=0")->select();
		// var_dump($all_members);exit();
		$all_vip = $db->field("count('*') as sum")
		->where("vip=1")->select();

		// var_dump($all_members);
		$this->assign('start',$start); //开始时间
		$this->assign('end', $end); //结束时间
		//week 本周 
		$this->assign('norm_sum', $res_normal[0]['sum']);//普通用户
		$this->assign('vip_sum', $res_vip[0]['sum']); //会员
		$this->assign('all1', $res_normal[0]['sum']+$res_vip[0]['sum']); //所有 总计
		
		//for month 月
		$this->assign('res_mod',$res_month[0]['sum']); //normal user 
		$this->assign('res_mod2',$res_month2[0]['sum']); //vip user

		$total_m = $res_month[0]['sum']+ $res_month2[0]['sum'];
		$this->assign('total_m',$total_m); //总计/本月新增数 

		$this->assign('begin_month',$begin_month_res); //月开始时间
		$this->assign('end_month', $end_month_res); //月结束时间

		$this->assign('rate',($res_normal[0]['sum']+$res_vip[0]['sum'])/($res_month[0]['sum']+$res_month2[0]['sum'])*100 .'%');
		
		if ($res_month[0]['sum']==0) {
		 	$this->assign('norm_rate','-');
		 } else{
		 	$this->assign('norm_rate', $res_normal[0]['sum']/$res_month[0]['sum']*100 .'%');
		 }
		if ($res_month2[0]['sum']==0) {
			$this->assign('vip_rate','-');
		}else{
			$this->assign('vip_rate', $res_vip[0]['sum']/$res_month2[0]['sum']*100 .'%');

		}

		$this->assign('all_members', $all_members[0]['sum']); //普通用户
		$this->assign('all_vip', $all_vip[0]['sum']); //会员

		$all_users = $all_members[0]['sum']+ $all_vip[0]['sum'];
		$this->assign('all_users', $all_users );//总计/ 总会员数/

		$mem = $all_members[0]['sum']; 
		$vip = $all_vip[0]['sum']; 
		$res = $res_month[0]['sum'];

		if($res == 0){
			$rate_month ='-';
		}
		else{
			$rate_month=round($res/$mem *100,2).'%';

		}
		$this->assign('rate_month',$rate_month); //普通用户 /月/百分比
		
		$res2 = $res_month2[0]['sum'];

		if ($res2 == 0) {
			$rate_vip = '-';
		}
		else{

			$rate_vip = $vip/($res2)*100 . '%';
		}
		$this->assign('rate_vip',$rate_vip); //会员 月/百分比

		$total_rate=round($total_m/$all_users*100,2).'%';
	
		
		$this->assign('total_rate', $total_rate);
		$this->display();

	}
	
}

?>
</pre>
