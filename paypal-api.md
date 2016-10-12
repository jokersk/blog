---
title: paypal API
date: 2016-10-07 17:34:27
tags:

---

## 這裡記錄一下 paypal 的 API, 讓以後需要用到的時候可以查到。

paypal 有3種比較常用的api 方法 
* **REST Api**
* **NVP Api**
* **HTML**

---

### REST :
這裡先說流程


代碼最重要:

*這是取得token的方法*


		public function gettoken()
		{


			$ch = curl_init();
			$clientId = "your client id";
			$secret = "your secret";

			curl_setopt($ch, CURLOPT_URL, "https://api.sandbox.paypal.com/v1/oauth2/token");
			curl_setopt($ch, CURLOPT_HEADER, false);
			curl_setopt($ch, CURLOPT_SSL_VERIFYHOST,0);
			curl_setopt($ch, CURLOPT_SSL_VERIFYPEER,0);
			curl_setopt($ch, CURLOPT_POST, true);
			curl_setopt($ch, CURLOPT_RETURNTRANSFER, true); 
			curl_setopt($ch, CURLOPT_USERPWD, $clientId.":".$secret);
			curl_setopt($ch, CURLOPT_POSTFIELDS, "grant_type=client_credentials");
			curl_setopt($ch, CURLOPT_SSLVERSION, 6);

			$result = curl_exec($ch);

			if(empty($result))die(curl_error($ch));
			else
			{
				$json = json_decode($result);

				$return  =  $json->access_token;
			}

			curl_close($ch);
			return $return;

		}

---
*這裡先創建一個payment*



		public function rest_api(){
			$token = $this->gettoken();
			$payment_json = '{
				"intent": "sale",
				"payer": {
					"payment_method": "paypal"
				},
				"redirect_urls": {
					"return_url": "http://return_URL_here",
					"cancel_url": "http://cancel_URL_here"
				},
				"transactions": [
				{
					"amount": {
						"total": "8.00",
						"currency": "HKD",
						"details": {
							"subtotal": "6.00",
							"tax": "1.00",
							"shipping": "1.00"
						}
					},
					"description": "This is payment description.",
					"item_list": { 
						"items":[
						{
							"quantity":"3", 
							"name":"Hat", 
							"price":"2.00",  
							"sku":"product12345", 
							"currency":"HKD"
						}
						]
					}
				}
				]
			}';


			$ch = curl_init();

			curl_setopt($ch, CURLOPT_URL, "https://api.sandbox.paypal.com/v1/payments/payment");
			curl_setopt($ch, CURLOPT_HEADER, false);
			curl_setopt($ch, CURLOPT_SSL_VERIFYHOST,0);
			curl_setopt($ch, CURLOPT_SSL_VERIFYPEER,0);
			curl_setopt($ch, CURLOPT_POST, true);
			curl_setopt($ch, CURLOPT_RETURNTRANSFER, true); 
			curl_setopt($ch, CURLOPT_HTTPHEADER, array(
				'Authorization: Bearer '.$token,
				'Accept: application/json',
				'Content-Type: application/json'
				));
			curl_setopt($ch, CURLOPT_POSTFIELDS, $payment_json);
			curl_setopt($ch, CURLOPT_SSLVERSION, 6);

			$result = curl_exec($ch);
			curl_close($ch);
			
			$array = json_decode($result);
			// var_dump($array);
			$redirect_urls = $array->links['1']->href;
			header("location: ".$redirect_urls); 

		}

裡面的 
		
		$token = $this->gettoken();

就是用了上面的 get token 方法取的那個token 

其中 

		$redirect_urls = $array->links['1']->href;


這裡會因為創建這個payment 之後 paypal 會 return 一串 json 來， 其中包括大概這樣子的東西： 
		
		"links":[
	    {
	      "href":"https://api.sandbox.paypal.com/v1/payments/payment/PAY-6RV70583SB702805EKEYSZ6Y",
	      "rel":"self",
	      "method":"GET"
	    },
	    {
	      "href":"https://www.sandbox.paypal.com/webscr?cmd=_express-checkout&token=EC-60U79048BN7719609",
	      "rel":"approval_url",
	      "method":"REDIRECT"
	    },
	    {
	      "href":"https://api.sandbox.paypal.com/v1/payments/payment/PAY-6RV70583SB702805EKEYSZ6Y/execute",
	      "rel":"execute",
	      "method":"POST"
	    }
	  	]

一個 "link" 的array. 我們要的就是中間的  https://www.sandbox.paypal.com/webscr?cmd=_express-checkout&token=EC-60U79048BN7719609
然後我們用 

		header("location: ".$redirect_urls); 

跳到這里，這時候其實就去了付款的頁面了 
