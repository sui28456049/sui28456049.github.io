---
title: CI框架多条件搜索
date: 2017-06-29 21:07:44
tags: php
category: php
---
> 多条件搜索关键在于sql语句的limit以及结果集数量的统计,这里我是用ci框架.其他框架大同小异.

### 控制器层(Controller)
```php
public function customer_mange($offset = 0)
    {
        $condition = [];
        $per_page = 6;
        $like = '';
       // 搜索
        if ($this->input->get('query')) {
            $condition['like'] = [
                ($this->data['type'] = $this->input->get('type'))
                =>
                ($this->data['query'] = $this->input->get('query'))
            ];
        }
       //意向查询
        if ($this->input->get('customer_intention') !== '') {
            $condition['customer_intention'] = $this->input->get('customer_intention');
        }
       //时间查询
        if ($this->input->get('timerange')) {
            $condition['timerange'] = $this->input->get('timerange');
        } else {
        //默认搜索30天
            $condition['timerange'] = $_GET['timerange'] = date('Y-m-d', strtotime('+30 day')) . '~' . date('Y-m-d');
        }

        $this->data['pagination'] = $this->get_page(
            'customer/customer_mange/',
            4,
            $this->m_customer->count($condition),
            $per_page
        );
        $this->data['userinfo'] = $this->m_customer->get_list($offset,$per_page,$condition);
        $this->data += $_GET;
        //意向
        $this->data['statuses'] = $this->m_customer->customer_intention;
        #sui($this->datas); exit;
        return $this->load->view('customer/kehu_list',$this->data);

    }
```

### 模型层(Model)
```php
  class M_customer extends CI_Model
    {
      public $customer_intention = [
            '2' => '无意向',
            '1' => '一般',
            '3' => '较好',
            '4' => '非常好'
        ];
    //查找所有数据
        public function get_list($offset='',$limit = 5, $condition = []){
            //用户意向
            if (isset($condition['customer_intention'])) {
                $condition['cd.customer_intention'] = $condition['customer_intention'];
                unset($condition['customer_intention']);
            } else {
                $this->db->where_in('cd.customer_intention',['1','2','3','4']);
                unset($condition['customer_intention']);
            }

            // 时间区间
               if (isset($condition['timerange'])) {
                   $timerange = explode('~', $condition['timerange']);
                   $time_start = date('Y-m-d H:i:s', strtotime($timerange[1]));
                   $time_end = date('Y-m-d H:i:s', strtotime($timerange[0]));
                   $this->db->where("cr.remind_time between '{$time_start}' and '{$time_end}'", null, false);
                   unset($condition['timerange']);
               }
            //搜索
               if (isset($condition['like'])) {
                    $like = $condition['like'];
                    unset($condition['like']);
                }
            $this->db->select('cd.*,cr.id cid,cr.remind_time remind_time')
                     ->from('customer_detail cd')
                     ->join('customer_remind cr', 'cd.id = cr.customer_id', 'left')
                     ->where($condition)
                     ->group_by('cd.id')
                     ->order_by('cd.id desc')
                     ->limit($limit, $offset);
            if (isset($like)) 
            {
              $this->db->like($like);
            }    
            $data = $this->db->get()->result_array();
            #sui($this->db->last_query());
            return $data;
            }

    public function count($condition = [])
        {
          //用户意向
          if (isset($condition['customer_intention'])) {
              $condition['cd.customer_intention'] = $condition['customer_intention'];
              unset($condition['customer_intention']);
          } else {
              $this->db->where_in('cd.customer_intention',['1','2','3','4']);
               unset($condition['customer_intention']);
          }

          // 时间区间
          if (isset($condition['timerange'])) {
              $timerange = explode('~', $condition['timerange']);
              $time_start = date('Y-m-d H:i:s', strtotime($timerange[1]));
              $time_end = date('Y-m-d H:i:s', strtotime($timerange[0]));
              $this->db->where("cr.remind_time between '{$time_start}' and '{$time_end}'", null, false);
              unset($condition['timerange']);
          }
          //搜索
          if (isset($condition['like'])) {
            $like = $condition['like'];
            unset($condition['like']);
          }

       $this->db->select('cd.*,cr.id cid,cr.remind_time remind_time')
                 ->from('customer_detail cd')
                 ->join('customer_remind cr', 'cd.id = cr.customer_id', 'left')
                 ->where($condition)
                 ->group_by('cd.id')
                 ->order_by('cd.id desc');
        //是否有模糊搜索
        if (isset($like)) 
            {
              $this->db->like($like);
            }    
        #sui($this->db);exit;
       return $this->db->count_all_results();
        }
    }
```

### view(视图层)
```php  
<form class="app-search">
            <dl>
                <dt class="col-1">
                    <select name="type">
                        <option value="cd.weixin">微信号码</option>
                        <option value="cd.name">用户姓名</option>
                        <option value="cd.qq">QQ号码</option>
                        <option value="cd.birthday">出生年月</option>           
                    </select>
                </dt>
                <dd class="col-10">
                    <div class="col-7">
                        <input name="query" class="col-6 txt-autotip">
                        <input type="submit" value="搜索" class="btn col-2">
                    </div>
                </dd>
            </dl>
            <dl>
                <dt class="col-1">提醒时间：</dt>
                <dd class="col-11">
              <input name="timerange" id="timerange" class="txt-datarange">
              <input type="submit" value="确定" class="btn">
                    <span>&nbsp; &nbsp; &nbsp; &nbsp;</span>
        <a href="<?php echo SITE_ADMIN_URL . '/customer/customer_mange?' . query_string('timerange') . '&timerange=' . date('Y-m-d') . '~' . date('Y-m-d', strtotime('+1 day')); ?>" <?php echo isset($timerange) && $timerange == date('Y-m-d') . '~' . date('Y-m-d', strtotime('+1 day')) ? 'class="now"' : ''; ?>>今天</a>

        <a href="<?php echo SITE_ADMIN_URL . '/customer/customer_mange?' . query_string('timerange') . '&timerange=' . date('Y-m-d', strtotime('+3 day')) . '~' . date('Y-m-d'); ?>" <?php echo isset($timerange) && $timerange == date('Y-m-d', strtotime('+3 day')) . '~' . date('Y-m-d') ? 'class="now"' : ''; ?>>最近3天</a>
        <a href="<?php echo SITE_ADMIN_URL . '/customer/customer_mange?' . query_string('timerange') . '&timerange=' . date('Y-m-d', strtotime('+7 day')) . '~' . date('Y-m-d'); ?>" <?php echo isset($timerange) && $timerange == date('Y-m-d', strtotime('+7 day')) . '~' . date('Y-m-d') ? 'class="now"' : ''; ?>>7天内</a>
         <a href="<?php echo SITE_ADMIN_URL . '/customer/customer_mange?' . query_string('timerange') . '&timerange=' . date('Y-m-d', strtotime('+15 day')) . '~' . date('Y-m-d'); ?>" <?php echo isset($timerange) && $timerange == date('Y-m-d', strtotime('+15 day')) . '~' . date('Y-m-d') ? 'class="now"' : ''; ?>>15天内</a>

                </dd>
            </dl>
            <dl>
                <dt class="col-1">客户意向：</dt>
                <dd class="col-11">
                    <a href="<?php echo SITE_ADMIN_URL . '/customer/customer_mange?' . query_string('customer_intention'); ?>" <?php echo !isset($customer_intention) ? 'class="now"' : ''; ?>>全部</a>

                    <?php foreach ($statuses as $k => $s): ?>
                    <a href="<?php echo SITE_ADMIN_URL . '/customer/customer_mange?' . query_string('customer_intention') . '&customer_intention=' . $k; ?>"
                     <?php echo isset($customer_intention) && $customer_intention == $k ? 'class="now"' : ''; ?>><?php echo $s; ?></a>
                    <?php endforeach; ?>
                </dd>
            </dl>
    </form>
```
### 视图层所用函数

```php
/**
 * 生成 query_string
 * @method query_string
 * @param  array/string       $exclude 需要排除的键
 * @return string
 */
function query_string($exclude = null)
{
    if (is_null($exclude)) {
        return $_SERVER['QUERY_STRING'];
    }

    parse_str($_SERVER['QUERY_STRING'], $query_array);

    $query_string = '';
    foreach ($query_array as $k => $v) {
        if (
            (is_array($exclude) && in_array($k, $exclude))
            or (is_string($exclude) && $k == $exclude)
        ) {
            unset($query_array[$k]);
        } else {
            $query_string .= "{$k}={$v}&";
        }
    }

    return substr($query_string, 0, -1);
}
```

## 以上就是通用的分页多条件搜索,如有疑问,可加QQ:28456049.      
