composer require lovecn/alipaybatchphp

[支付宝转账api](https://doc.open.alipay.com/doc2/detail?treeId=64&articleId=103569&docType=1)

以下代码采用laravel框架，如果原生php自行改动

###同步请求
```js
    /**
     * 支付宝转账
     * @param  string $batch_no    转账批次号
     * @param  string $batch_fee   付款总金额
     * @param  string $batch_num   付款总笔数
     * @param  string $detail_data 付款详细数据
     * @return string              跳转支付宝
     */
    public function alipay($batch_no, $batch_fee, $batch_num, $detail_data)
    {
        
        $notify_url = config('services.alipay.notify');;//异步回调地址
        $email = config('services.alipay.email');
        $account_name = config('services.alipay.account');
        $pay_date = date('Ymd');
        /**
        测试
        $batch_no = date('Ymd').rand(1000000,9999999999);
        $batch_fee = 0.02;
        $batch_num = 2;
        $fee = 0.01;
        $detail_data = date('Ymd').rand(1000000,9999999999).'^xxx@qq.com^小明^'.$fee.'^转账|'.date('Ymd').rand(1000000,9999999999).'^xxx@qq.com^小红^'.$fee.'^转账';
        */
        
        //合作身份者id，以2088开头的16位纯数字
        $alipay_config['partner']       = config('services.alipay.id');
        
        //安全检验码，以数字和字母组成的32位字符
        $alipay_config['key']           = config('services.alipay.key');
        
        //签名方式 不需修改
        $alipay_config['sign_type']    = strtoupper('MD5');
        
        //字符编码格式 目前支持 gbk 或 utf-8
        $alipay_config['input_charset']= strtolower('utf-8');
        
        //ca证书路径地址，用于curl中ssl校验
        //请保证cacert.pem文件在当前文件夹目录中
        $alipay_config['cacert']    = getcwd().'\\cacert.pem';
        
        //访问模式,根据自己的服务器是否支持ssl访问，若支持请选择https；若不支持请选择http
        $alipay_config['transport']    = 'http';
        
        //构造要请求的参数数组，无需改动
        $parameter = array(
            "service" => "batch_trans_notify",
            "partner" => trim($alipay_config['partner']),
            "notify_url"    => $notify_url,
            "email" => $email,
            "account_name"  => $account_name,
            "pay_date"  => $pay_date,
            "batch_no"  => $batch_no,
            "batch_fee" => $batch_fee,
            "batch_num" => $batch_num,
            "detail_data"   => $detail_data,
            "_input_charset"    => trim(strtolower($alipay_config['input_charset']))
        );
        //建立请求
        $alipaySubmit = new \AlipaySubmit($alipay_config);
        $html_text = $alipaySubmit->buildRequestForm($parameter,"get", "确认");

        return $html_text;
    }
```
###异步回调
```js
public function postAlipay()
    {
        $success = Input::get('success_details', '');
        $fail = Input::get('fail_details', '');
        $notifyId = Input::get('notify_id', '');
        $batchNo = Input::get('batch_no', '');
        $alipayConfig = [
            'partner' => config('services.alipay.id'),
            'key' => config('services.alipay.key'),
            'sign_type' => strtoupper('MD5'),
            'input_charset' => strtolower('utf-8'),
            'cacert' => getcwd() . '\\cacert.pem',
            'transport' => 'http',
        ];
        $alipayNotify = new \AlipayNotify($alipayConfig);
        $result = $alipayNotify->verifyNotify();
        if ($result) {
            //更新订单状态
            return 'success';
        }

        return 'fail';
    }
```
