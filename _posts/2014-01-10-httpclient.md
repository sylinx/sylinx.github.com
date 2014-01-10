---
layout: post
title: httpclient优化
category: default
---
h3. httpclient优化

社交圈跟QB接口采用的是http方式，目前使用httpclient4.0发起请求，未采用池化管理，每次请求重新new 个httpclient。特地做了如下测试验证下两者差别，发现单线程环境下采用池化效率大概是未采用池化的两倍，多线程环境大概是3倍多，随着并发量增多，效率应该更明显

线程池参数

{code}
private static int MAX_CONNECTIONS = 200;
private static int MAX_ROUTE_CONNECTIONS = 30;
{code}



h3.


h3. 单线程访问100次总耗时对比（单位s）

|| 未池化 || 池化 ||
| 25.886 | 12.692 |
| 23.301 | 11.369 |
| 26.695 | 11.073 |
| 24.852 | 13.085 |
| 24.985 | 11.008 |



h3. 多线程 10个线程每个线程执行50次对比（单位s）

|| 未池化 || 池化 ||
| 1.549 seconds \\
1.562 seconds \\
1.563 seconds \\
1.545 seconds \\
1.564 seconds \\
1.915 seconds \\
2.164 seconds \\
2.454 seconds \\
6.089 seconds \\
8.622 seconds \\
total 29.027 seconds | 0.864 seconds \\
0.864 seconds \\
0.874 seconds \\
0.874 seconds \\
0.874 seconds \\
0.874 seconds \\
0.874 seconds \\
1.044 seconds \\
1.044 seconds \\
1.051 seconds \\
total 9.237 seconds |
| 1.414 seconds \\
1.427 seconds \\
1.427 seconds \\
1.429 seconds \\
1.639 seconds \\
1.639 seconds \\
4.542 seconds \\
5.646 seconds \\
5.891 seconds \\
5.891 seconds \\
total 30.946 seconds | 0.884 seconds \\
0.889 seconds \\
0.891 seconds \\
0.895 seconds \\
0.899 seconds \\
0.900 seconds \\
0.904 seconds \\
0.906 seconds \\
1.108 seconds \\
1.107 seconds \\
total 9.383 seconds |
| 1.308 seconds \\
1.315 seconds \\
1.326 seconds \\
1.335 seconds \\
1.335 seconds \\
1.336 seconds \\
1.347 seconds \\
3.119 seconds \\
3.413 seconds \\
6.485 seconds \\
total 22.318 seconds | 0.914 seconds \\
0.920 seconds \\
0.921 seconds \\
0.921 seconds \\
0.922 seconds \\
0.963 seconds \\
0.964 seconds \\
0.963 seconds \\
0.968 seconds \\
0.969 seconds \\
total 9.424 seconds |
| 1.382 seconds \\
1.395 seconds \\
1.395 seconds \\
1.395 seconds \\
1.395 seconds \\
1.396 seconds \\
1.464 seconds \\
5.568 seconds \\
7.335 seconds \\
7.335 seconds \\
total 30.060 seconds | 0.873 seconds \\
0.881 seconds \\
0.882 seconds \\
0.883 seconds \\
0.885 seconds \\
0.888 seconds \\
0.888 seconds \\
0.889 seconds \\
0.894 seconds \\
1.280 seconds \\
total 9.244 seconds |
| 1.669 seconds \\
1.679 seconds \\
1.680 seconds \\
1.685 seconds \\
1.686 seconds \\
1.749 seconds \\
1.750 seconds \\
5.865 seconds \\
6.112 seconds \\
6.113 seconds \\
total 29.989 seconds | 0.887 seconds \\
0.897 seconds \\
0.898 seconds \\
0.907 seconds \\
0.908 seconds \\
0.908 seconds \\
0.908 seconds \\
0.916 seconds \\
0.915 seconds \\
0.920 seconds \\
total 9.063 seconds |



h3. 多线程 50个线程每个线程执行50次对比（单位s）

|| 未池化 || 池化 ||
| 1.237 seconds \\
1.257 seconds \\
1.264 seconds \\
1.272 seconds \\
1.549 seconds \\
1.747 seconds \\
1.820 seconds \\
1.831 seconds \\
2.095 seconds \\
2.099 seconds \\
2.109 seconds \\
2.284 seconds \\
2.393 seconds \\
3.101 seconds \\
3.392 seconds \\
4.270 seconds \\
4.328 seconds \\
4.330 seconds \\
4.335 seconds \\
4.336 seconds \\
4.336 seconds \\
4.363 seconds \\
4.412 seconds \\
4.426 seconds \\
4.484 seconds \\
4.578 seconds \\
4.590 seconds \\
4.622 seconds \\
4.702 seconds \\
4.704 seconds \\
4.870 seconds \\
4.907 seconds \\
4.932 seconds \\
4.935 seconds \\
4.932 seconds \\
4.947 seconds \\
5.583 seconds \\
6.987 seconds \\
7.224 seconds \\
7.225 seconds \\
7.235 seconds \\
7.250 seconds \\
7.265 seconds \\
7.263 seconds \\
7.264 seconds \\
7.289 seconds \\
7.291 seconds \\
7.353 seconds \\
7.590 seconds \\
11.221 seconds \\
total 229.829 seconds | 1.262 seconds \\
1.276 seconds \\
1.277 seconds \\
1.286 seconds \\
1.295 seconds \\
1.313 seconds \\
1.330 seconds \\
1.343 seconds \\
1.342 seconds \\
1.343 seconds \\
1.350 seconds \\
1.354 seconds \\
1.356 seconds \\
1.357 seconds \\
1.357 seconds \\
1.360 seconds \\
1.362 seconds \\
1.363 seconds \\
1.365 seconds \\
1.364 seconds \\
1.367 seconds \\
1.370 seconds \\
1.371 seconds \\
1.372 seconds \\
1.371 seconds \\
1.376 seconds \\
1.378 seconds \\
1.378 seconds \\
1.362 seconds \\
1.380 seconds \\
1.381 seconds \\
1.381 seconds \\
1.372 seconds \\
1.389 seconds \\
1.390 seconds \\
1.392 seconds \\
1.394 seconds \\
1.395 seconds \\
1.398 seconds \\
1.399 seconds \\
1.399 seconds \\
1.400 seconds \\
1.403 seconds \\
1.405 seconds \\
1.411 seconds \\
1.411 seconds \\
1.416 seconds \\
1.421 seconds \\
1.424 seconds \\
1.808 seconds \\
total 68.765 seconds |
| 1.250 seconds \\
1.269 seconds \\
1.267 seconds \\
1.279 seconds \\
1.284 seconds \\
1.665 seconds \\
1.695 seconds \\
1.705 seconds \\
1.722 seconds \\
1.778 seconds \\
2.136 seconds \\
2.639 seconds \\
2.638 seconds \\
2.789 seconds \\
4.568 seconds \\
4.589 seconds \\
4.591 seconds \\
4.595 seconds \\
4.628 seconds \\
4.628 seconds \\
4.678 seconds \\
4.704 seconds \\
4.704 seconds \\
4.704 seconds \\
5.617 seconds \\
5.660 seconds \\
5.711 seconds \\
5.803 seconds \\
6.143 seconds \\
7.213 seconds \\
7.215 seconds \\
7.217 seconds \\
7.237 seconds \\
7.235 seconds \\
7.237 seconds \\
7.238 seconds \\
7.245 seconds \\
7.250 seconds \\
7.253 seconds \\
7.260 seconds \\
7.263 seconds \\
7.277 seconds \\
7.294 seconds \\
7.820 seconds \\
7.880 seconds \\
7.885 seconds \\
7.936 seconds \\
7.938 seconds \\
8.244 seconds \\
9.227 seconds \\
total 258.807 seconds | 1.412 seconds \\
1.440 seconds \\
1.444 seconds \\
1.449 seconds \\
1.459 seconds \\
1.457 seconds \\
1.464 seconds \\
1.471 seconds \\
1.476 seconds \\
1.485 seconds \\
1.488 seconds \\
1.488 seconds \\
1.492 seconds \\
1.494 seconds \\
1.496 seconds \\
1.497 seconds \\
1.505 seconds \\
1.503 seconds \\
1.505 seconds \\
1.504 seconds \\
1.511 seconds \\
1.515 seconds \\
1.513 seconds \\
1.523 seconds \\
1.523 seconds \\
1.526 seconds \\
1.528 seconds \\
1.531 seconds \\
1.531 seconds \\
1.534 seconds \\
1.535 seconds \\
1.534 seconds \\
1.535 seconds \\
1.535 seconds \\
1.538 seconds \\
1.537 seconds \\
1.539 seconds \\
1.541 seconds \\
1.544 seconds \\
1.547 seconds \\
1.545 seconds \\
1.548 seconds \\
1.549 seconds \\
1.549 seconds \\
1.556 seconds \\
1.561 seconds \\
1.561 seconds \\
1.565 seconds \\
1.567 seconds \\
1.742 seconds \\
total 75.891 seconds |


{% include references.md %}