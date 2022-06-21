<!--ts-->
* [php](#php)
   * [increment order number](#increment-order-number)
   * [random string](#random-string)
   * [data chunk](#data-chunk)
   * [ip region](#ip-region)
   * [sign data with hmac](#sign-data-with-hmac)
   * [microsecond](#microsecond)
   * [random amount](#random-amount)
   * [json](#json-decode)
   * [jwt](#jwt)
   * [export excel](#export-excel)
<!--te-->

# php

## array_map

回调只有一个值，没有键，多个数组参数，回调就有多个值的参数

```php
dump(array_map(function ($val) {
    return $val * 2;
}, array_flip([4, 5, 6])));
/*
array:3 [▼
  4 => 0
  5 => 2
  6 => 4
]
*/
```
## array_walk

```php
$foobar = array_flip([4, 5, 6]);
array_walk($foobar, function (&$v, $k) {
    $v = $v + $k;
});
dump($foobar);
/*
array:3 [▼
  4 => 4
  5 => 6
  6 => 8
]
*/
```
## increment order number

  `date('ymd') . sprintf('%010d', mt_rand(10000, 99999))`
  
  配合redis的incr

## random string

  `md5(uniqid(mt_rand(), true))`

## data chunk

  `collect($dataRows)->chunk(500)->each(function ($rows) {})`

## ip region

  https://github.com/lionsoul2014/ip2region

  `$ip2region->binarySearch($ip)`

## sign data with hmac

  ```php
  $iv = openssl_random_pseudo_bytes(16);
  $encrypted = openssl_encrypt($payload, 'aes-128-cbc', $encryptKey, 0, $iv);
  $mac = hash_hmac('sha256', base64_encode($iv) . $encrypted, $hashKey, 1);
  $signed = base64_encode($mac) . base64_encode($iv) . $encrypted;
  
  if (hash_equals(base64_encode(hash_hmac('sha256', substr($signed, 44), $hashKey, 1)), substr($signed, 0, 44))) {
      $b64iv = substr($signed, 44, 24);
      $encrypted = substr($signed, 68);
      $decrypted = openssl_decrypt($encrypted, 'aes-128-cbc', $encryptKey, 0, base64_decode($b64iv));
  }
  
  # url safe
  str_replace(['/', '+', '='], ['_', '-', ''], $base64);
  str_replace(['_', '-'], ['/', '+'], $base64);
  ```
  
## microsecond

  `(new \DateTime())->format('u')`
  
  `substr(microtime(), 2, 6)`
  
## random amount

  ```php
  while ($leftBonus > 0) {
      $randBonus = mt_rand(1, min($leftBonus, 100));
      $leftBonus -= $randBonus;
      $idx = mt_rand(0, $count - 1);
      $bonusArr[$idx] = $bonusArr[$idx] + $randBonus;
  }
  ```

## json
  
  json_encode 常用选项JSON_UNESCAPED_UNICODE（256），保留汉字；JSON_UNESCAPED_SLASHES（64），保留`/`，不加会转义路径分隔符为`\/`。合起来就是320
  
  `json_decode(file_get_contents('php://input'), true)`
  
## jwt

  ```php
  use Firebase\JWT\JWT;
  use GuzzleHttp\Client;
  
//  参考：League\OAuth2\Server\Entities\Traits\AccessTokenTrait
  $jwtArr = [
      "aud" => "aud",
      "iss" => "iss",
//      "sub" => "subject",
//      "jti" => "java token id",
      "iat" => time(),
      "nbf" => time(),
      "exp" => time() + 600,
      "data" => 'foobar'
  ];
  $jwt = JWT::encode($jwtArr, $key);
//  参考：EasyWeChat\Kernel\BaseClient
  $client = new Client(['base_uri' => 'https://foobar.com/api/']);
  $response = $client->post('foo/bar', [
      'form_params' => [
          'jwt' => $jwt
      ],
//      'connect_timeout' => 30,
      'timeout' => 30,
//      'read_timeout' => 30,
  ]);
  $result = json_decode((string) $response->getBody(), true);
  
  //try catch
  $requestData = (array)JWT::decode($_POST['jwt'], $key, array('HS256'));
  ```

## export excel

  ```php
  use PhpOffice\PhpSpreadsheet\Spreadsheet;
  use PhpOffice\PhpSpreadsheet\Writer\Xlsx;
  
  $spreadsheet = new Spreadsheet();
  $sheet = $spreadsheet->getActiveSheet();
  $sheet->getColumnDimension('A')->setWidth(20.0);
  $sheet->getColumnDimension('B')->setWidth(20.0);
  $sheet->setCellValue('A1', '姓名');
  $sheet->setCellValue('B1', '手机');
  
  $sheet->setCellValue('A2', $name);
  $sheet->setCellValue('B2', $mobile);
  
  header('Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet');
  header('Content-Disposition: attachment;filename="' . date('YmdHis') . '.xlsx"');
  header('Cache-Control: max-age=0');
  
  $writer = new Xlsx($spreadsheet);
  $writer->save('php://output');
  ```
