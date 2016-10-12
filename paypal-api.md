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

*先取得token*

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
