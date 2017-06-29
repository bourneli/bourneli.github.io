---
layout: post
title:  Excel中的矩阵乘法
categories: [office,excel]
---

Excel中的矩阵乘法和现实的矩阵乘法一样，需要两个矩阵的行列兼容，即A矩阵的列需要和B矩阵的行相同，在一个空的cell中输入`=mmult(A,B)`。但是你以为如果这样，就得到了结果矩阵，那就错了。这样只会结果矩阵的第一个值。如果不展开，是得不到结果矩阵的。

展开的正确姿势，

1. 用鼠标选取一片区域，该区域左上角第一个cell输入`=mmult(A,B)`，该区域的大小于结果矩阵行列相同。
2. 按**F2**，左上角公式会展开，相关矩阵区域会高亮（参考下图）
3. 最后联合按ctr+shift+enter，搞定！

<div align='center'>
  <img src='/img/excel_mmult_demo.png' />
</div>

最后数据就会落到步骤1选取的指定区域。
<div align='center'>
  <img src='/img/excel_mmult_result.png' />
</div>
