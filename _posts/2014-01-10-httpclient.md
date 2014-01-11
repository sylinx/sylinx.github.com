---
layout: post
title: HttpClient优化
category: default
---


社交圈跟QB接口采用的是http方式，目前使用httpclient4.0发起请求，未采用池化管理，每次请求重新new 个httpclient。特地做了如下测试验证下两者差别，发现单线程环境下采用池化效率大概是未采用池化的两倍，多线程环境大概是3倍多，随着并发量增多，效率应该更明显

线程池参数

`` 
private static int MAX_CONNECTIONS = 200; 
private static int MAX_CONNECTIONS_PER_ROUTE = 30;
`` 




单线程访问100次总耗时对比（单位s）
----------

<table>
 <tr >
  <td >未池化</td>
  <td >池化</td>
 </tr>
  <tr >
  <td >25.886</td>
  <td >12.692</td>
 </tr>
  <tr >
  <td >23.301</td>
  <td >11.369</td>
 </tr>
  <tr >
  <td >26.695</td>
  <td >11.073</td>
 </tr>
  <tr >
  <td >24.852</td>
  <td >13.085</td>
 </tr>
  <tr >
  <td >24.985</td>
  <td >11.008</td>
 </tr>
 </table>



多线程 10个线程每个线程执行50次对比（单位s）
--------------

<table>
 <tr >
  <td >未池化</td>
  <td >池化</td>
 </tr>
 <tr >
  <td >1.549
  seconds<br>
  1.562 seconds<br>
  1.563 seconds<br>
  1.545 seconds<br>
  1.564 seconds<br>
  1.915 seconds<br>
  2.164 seconds<br>
  2.454 seconds<br>
  6.089 seconds<br>
  8.622 seconds<br>
  total 29.027 second</td>
  <td >0.864 seconds <br>
  0.864 seconds <br>
  0.874 seconds <br>
  0.874 seconds <br>
  0.874 seconds <br>
  0.874 seconds <br>
  0.874 seconds <br>
  1.044 seconds <br>
  1.044 seconds <br>
  1.051 seconds <br>
  total 9.237 seconds</td>
 </tr>
 <tr>
  <td>1.414
  seconds <br>
  1.427 seconds <br>
  1.427 seconds <br>
  1.429 seconds <br>
  1.639 seconds <br>
  1.639 seconds <br>
  4.542 seconds <br>
  5.646 seconds <br>
  5.891 seconds <br>
  5.891 seconds <br>
  total 30.946 seconds</td>
  <td>0.884 seconds <br>
  0.889 seconds <br>
  0.891 seconds <br>
  0.895 seconds <br>
  0.899 seconds <br>
  0.900 seconds <br>
  0.904 seconds <br>
  0.906 seconds <br>
  1.108 seconds <br>
  1.107 seconds <br>
  total 9.383 seconds</td>
 </tr>
 <tr >
  <td>1.308
  seconds <br>
  1.315 seconds <br>
  1.326 seconds <br>
  1.335 seconds <br>
  1.335 seconds <br>
  1.336 seconds <br>
  1.347 seconds <br>
  3.119 seconds <br>
  3.413 seconds <br>
  6.485 seconds <br>
  total 22.318 seconds</td>
  <td>0.914 seconds <br>
  0.920 seconds <br>
  0.921 seconds <br>
  0.921 seconds <br>
  0.922 seconds <br>
  0.963 seconds <br>
  0.964 seconds <br>
  0.963 seconds <br>
  0.968 seconds <br>
  0.969 seconds <br>
  total 9.424 seconds</td>
 </tr>
 <tr >
  <td>1.382
  seconds <br>
  1.395 seconds <br>
  1.395 seconds <br>
  1.395 seconds <br>
  1.395 seconds <br>
  1.396 seconds <br>
  1.464 seconds <br>
  5.568 seconds <br>
  7.335 seconds <br>
  7.335 seconds <br>
  total 30.060 seconds</td>
  <td>0.873 seconds <br>
  0.881 seconds <br>
  0.882 seconds <br>
  0.883 seconds <br>
  0.885 seconds <br>
  0.888 seconds <br>
  0.888 seconds <br>
  0.889 seconds <br>
  0.894 seconds <br>
  1.280 seconds <br>
  total 9.244 seconds</td>
 </tr>
 <tr>
  <td>1.669
  seconds <br>
  1.679 seconds <br>
  1.680 seconds <br>
  1.685 seconds <br>
  1.686 seconds <br>
  1.749 seconds <br>
  1.750 seconds <br>
  5.865 seconds <br>
  6.112 seconds <br>
  6.113 seconds <br>
  total 29.989 seconds</td>
  <td>0.887 seconds <br>
  0.897 seconds <br>
  0.898 seconds <br>
  0.907 seconds <br>
  0.908 seconds <br>
  0.908 seconds <br>
  0.908 seconds <br>
  0.916 seconds <br>
  0.915 seconds <br>
  0.920 seconds <br>
  total 9.063 seconds</td>
 </tr>
</table>


多线程 50个线程每个线程执行50次对比（单位s）
--------------

<table >

 <tr >
  <td >未池化</td>
  <td>池化</td>
 </tr>
 <tr >
  <td>1.237
  seconds <br>
  1.257 seconds <br>
  1.264 seconds <br>
  1.272 seconds <br>
  1.549 seconds <br>
  1.747 seconds <br>
  1.820 seconds <br>
  1.831 seconds <br>
  2.095 seconds <br>
  2.099 seconds <br>
  2.109 seconds <br>
  2.284 seconds <br>
  2.393
  seconds <br>
  3.101 seconds <br>
  3.392 seconds <br>
  4.270 seconds <br>
  4.328 seconds <br>
  4.330 seconds <br>
  4.335 seconds <br>
  4.336 seconds <br>
  4.336 seconds <br>
  4.363 seconds <br>
  4.412 seconds <br>
  4.426 seconds <br>
  4.484
  seconds <br>
  4.578 seconds <br>
  4.590 seconds <br>
  4.622 seconds <br>
  4.702 seconds <br>
  4.704 seconds <br>
  4.870 seconds <br>
  4.907 seconds <br>
  4.932 seconds <br>
  4.935 seconds <br>
  4.932 seconds <br>
  4.947 seconds <br>
  5.583 seconds
  <br>
  6.987 seconds <br>
  7.224 seconds <br>
  7.225 seconds <br>
  7.235 seconds <br>
  7.250 seconds <br>
  7.265 seconds <br>
  7.263 seconds <br>
  7.264 seconds <br>
  7.289 seconds <br>
  7.291 seconds <br>
  7.353 seconds <br>
  7.590 seconds
  <br>
  11.221 seconds <br>
  total 229.829 seconds</td>
  <td>1.262 seconds <br>
  1.276 seconds <br>
  1.277 seconds <br>
  1.286 seconds <br>
  1.295 seconds <br>
  1.313 seconds <br>
  1.330 seconds <br>
  1.343 seconds <br>
  1.342 seconds <br>
  1.343 seconds <br>
  1.350 seconds <br>
  1.354 seconds <br>
  1.356
  seconds <br>
  1.357 seconds <br>
  1.357 seconds <br>
  1.360 seconds <br>
  1.362 seconds <br>
  1.363 seconds <br>
  1.365 seconds <br>
  1.364 seconds <br>
  1.367 seconds <br>
  1.370 seconds <br>
  1.371 seconds <br>
  1.372 seconds <br>
  1.371
  seconds <br>
  1.376 seconds <br>
  1.378 seconds <br>
  1.378 seconds <br>
  1.362 seconds <br>
  1.380 seconds <br>
  1.381 seconds <br>
  1.381 seconds <br>
  1.372 seconds <br>
  1.389 seconds <br>
  1.390 seconds <br>
  1.392 seconds <br>
  1.394 seconds
  <br>
  1.395 seconds <br>
  1.398 seconds <br>
  1.399 seconds <br>
  1.399 seconds <br>
  1.400 seconds <br>
  1.403 seconds <br>
  1.405 seconds <br>
  1.411 seconds <br>
  1.411 seconds <br>
  1.416 seconds <br>
  1.421 seconds <br>
  1.424 seconds
  <br>
  1.808 seconds <br>
  total 68.765 seconds</td>
 </tr>
 <tr>
  <td>1.250
  seconds <br>
  1.269 seconds <br>
  1.267 seconds <br>
  1.279 seconds <br>
  1.284 seconds <br>
  1.665 seconds <br>
  1.695 seconds <br>
  1.705 seconds <br>
  1.722 seconds <br>
  1.778 seconds <br>
  2.136 seconds <br>
  2.639 seconds <br>
  2.638
  seconds <br>
  2.789 seconds <br>
  4.568 seconds <br>
  4.589 seconds <br>
  4.591 seconds <br>
  4.595 seconds <br>
  4.628 seconds <br>
  4.628 seconds <br>
  4.678 seconds <br>
  4.704 seconds <br>
  4.704 seconds <br>
  4.704 seconds <br>
  5.617
  seconds <br>
  5.660 seconds <br>
  5.711 seconds <br>
  5.803 seconds <br>
  6.143 seconds <br>
  7.213 seconds <br>
  7.215 seconds <br>
  7.217 seconds <br>
  7.237 seconds <br>
  7.235 seconds <br>
  7.237 seconds <br>
  7.238 seconds <br>
  7.245 seconds
  <br>
  7.250 seconds <br>
  7.253 seconds <br>
  7.260 seconds <br>
  7.263 seconds <br>
  7.277 seconds <br>
  7.294 seconds <br>
  7.820 seconds <br>
  7.880 seconds <br>
  7.885 seconds <br>
  7.936 seconds <br>
  7.938 seconds <br>
  8.244 seconds
  <br>
  9.227 seconds <br>
  total 258.807 seconds</td>
  <td>1.412 seconds <br>
  1.440 seconds <br>
  1.444 seconds <br>
  1.449 seconds <br>
  1.459 seconds <br>
  1.457 seconds <br>
  1.464 seconds <br>
  1.471 seconds <br>
  1.476 seconds <br>
  1.485 seconds <br>
  1.488
  seconds <br>
  1.488 seconds <br>
  1.492 seconds <br>
  1.494 seconds <br>
  1.496 seconds <br>
  1.497 seconds <br>
  1.505 seconds <br>
  1.503 seconds <br>
  1.505 seconds <br>
  1.504 seconds <br>
  1.511 seconds <br>
  1.515 seconds <br>
  1.513
  seconds <br>
  1.523 seconds <br>
  1.523 seconds <br>
  1.526 seconds <br>
  1.528 seconds <br>
  1.531 seconds <br>
  1.531 seconds <br>
  1.534 seconds <br>
  1.535 seconds <br>
  1.534 seconds <br>
  1.535 seconds <br>
  1.535 seconds <br>
  1.538
  seconds <br>
  1.537 seconds <br>
  1.539 seconds <br>
  1.541 seconds <br>
  1.544 seconds <br>
  1.547 seconds <br>
  1.545 seconds <br>
  1.548 seconds <br>
  1.549 seconds <br>
  1.549 seconds <br>
  1.556 seconds <br>
  1.561 seconds <br>
  1.561 seconds
  <br>
  1.565 seconds <br>
  1.567 seconds <br>
  1.742 seconds <br>
  total 75.891 seconds</td>
 </tr>
</table>

{% include references.md %}