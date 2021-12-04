<p align="center"><img src="https://raw.githubusercontent.com/tquangsdh20/winedataset/master/.github/wine_logo.svg"></p>
<p align="center"><img src="https://img.shields.io/badge/Group5-Le%20Thai%20%7C%20Tran%20Quang%20%7C%20Le%20Thai%20Duy%20%7C%20Le%20Nhu%20Chien%20%7C%20Phan%20Van%20Trung-blue?style=plastic"> <img src="https://img.shields.io/badge/Lecturer-Nguyen%20Hoang%20Dzung-orange?style=plastic"> <img src="https://img.shields.io/badge/version-4.0.5-blue?style=plastic&logo=R"></p>

## 1. Thu thập dữ liệu (Data Collection)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Dữ liệu gồm có hai tập *datasets* về các mẫu rượu vang đỏ và trắng thu thập ở phía bắc của Bồ Đào Nha nhằm mô hình hóa chất lượng của rượu vang dựa trên các kiểm nghiệm hóa lý. Tập dữ liệu nghiêm cứu được lấy từ [UC Irvine Machine Learning Repository](https://archive.ics.uci.edu/ml/index.php) có thể tải về [tại đây](https://archive.ics.uci.edu/ml/machine-learning-databases/wine-quality/).

### 1.1. Mô tả dữ liệu
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Hai bộ dữ liệu trên liên quan đến các biến thể về rượu vang trắng và đỏ ở Bồ Đào Nha, tập dữ liệu về rượu vang trắng gồm có 4898 bản ghi (individuals) và rượu vang đỏ có 1599 bản ghi (individuals). Trong đó, dữ liệu phân tích gồm có 12 biến dữ liệu, trong đó có 11 biến là các thông số thành phần hóa lý và 1 biến đánh giá chất lượng.

### 1.2. Đặt vấn đề
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Bài toán được đặt ra trong nghiên cứu này liệu có thể tiên lượng được chất lượng của loại rượu dựa trên các thành phần hóa lý của nó hay không? Và màu sắc của rượu có phụ thuộc vào các thành phần hóa lý trên hay không?

## 2. Trình bày dữ liệu

Dữ liệu trong hai tập trên gồm 12 biến như sau:

- **fixed.acidity**: thông số đo độ chua cố định
- **volatile.acidity**: mức độ bay hơi của rượu
- **citric.acid**: nồng độ Axit Citric
- **residual.sugar**: lượng đường
- **chlorides**: nồng độ Clo
- **free.sulfur.dioxide**: các thành phần SO2 tự do
- **total.sulfur.dioxide**: tổng số gốc SO2
- **density**: mức độ đậm đặc của rượu
- **pH**: nồng độ pH
- **sulphates**: các gốc SO4
- **alcohol**: nồng độ cồn
- **quality**: kết quả chất lượng được đánh giá theo thang điểm từ 1-10


Đọc hai tập dữ liệu vào và tiến hành các bước tiền xử lý


```R
require(dplyr)
require(ggplot2)
require(rpart)
require(caret)
require(rpart.plot)
require(FactoMineR)
require(PerformanceAnalytics)
```


```R
set.seed(1)
red_wine = read.csv('winequality-red.csv',sep = ';')
white_wine = read.csv('winequality-white.csv',sep = ';')
white_wine = white_wine %>% sample_frac(0.3264)
```

Ghép hai tập dữ liệu lại và thêm một biến **type** để gắn nhãn cho các loại phân tích (*white và red*)


```R
wine_dat <- rbind(
    data.frame(white_wine,type='white')
    ,data.frame(red_wine,type='red')
)
wine_dat$type <- as.factor(wine_dat$type)
# wine_dat$quality <- as.factor(wine_dat$quality)
head(wine_dat)
```


<table class="dataframe">
<caption>A data.frame: 6 × 13</caption>
<thead>
	<tr><th></th><th scope=col>fixed.acidity</th><th scope=col>volatile.acidity</th><th scope=col>citric.acid</th><th scope=col>residual.sugar</th><th scope=col>chlorides</th><th scope=col>free.sulfur.dioxide</th><th scope=col>total.sulfur.dioxide</th><th scope=col>density</th><th scope=col>pH</th><th scope=col>sulphates</th><th scope=col>alcohol</th><th scope=col>quality</th><th scope=col>type</th></tr>
	<tr><th></th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;int&gt;</th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>6.8</td><td>0.37</td><td>0.47</td><td>11.2</td><td>0.071</td><td>44.0</td><td>136</td><td>0.99680</td><td>2.98</td><td>0.88</td><td> 9.2</td><td>5</td><td>white</td></tr>
	<tr><th scope=row>2</th><td>7.1</td><td>0.24</td><td>0.34</td><td> 1.2</td><td>0.045</td><td> 6.0</td><td>132</td><td>0.99132</td><td>3.16</td><td>0.46</td><td>11.2</td><td>4</td><td>white</td></tr>
	<tr><th scope=row>3</th><td>6.9</td><td>0.32</td><td>0.13</td><td> 7.8</td><td>0.042</td><td>11.0</td><td>117</td><td>0.99600</td><td>3.23</td><td>0.37</td><td> 9.2</td><td>5</td><td>white</td></tr>
	<tr><th scope=row>4</th><td>7.5</td><td>0.23</td><td>0.49</td><td> 7.7</td><td>0.049</td><td>61.0</td><td>209</td><td>0.99410</td><td>3.14</td><td>0.30</td><td>11.1</td><td>7</td><td>white</td></tr>
	<tr><th scope=row>5</th><td>8.6</td><td>0.36</td><td>0.26</td><td>11.1</td><td>0.030</td><td>43.5</td><td>171</td><td>0.99480</td><td>3.03</td><td>0.49</td><td>12.0</td><td>5</td><td>white</td></tr>
	<tr><th scope=row>6</th><td>7.7</td><td>0.28</td><td>0.63</td><td>11.1</td><td>0.039</td><td>58.0</td><td>179</td><td>0.99790</td><td>3.08</td><td>0.44</td><td> 8.8</td><td>4</td><td>white</td></tr>
</tbody>
</table>



## 3. Phân tích và giải thích dữ liệu

Trước khi chọn phương pháp phân tích ta kiểm tra lại phân bố về chất lượng của loại rượu được phân bố như thế nào

### 3.1. Biểu diễn phân bố chất lượng của rượu


```R
options(repr.plot.width = 7, repr.plot.height = 5, repr.plot.res = 200)
hist(wine_dat$quality,xlab='Quality',main='Histogram of Wine')
```


    
![png](./.github/output_10_0.png?sanitize=true)
    



```R
options(repr.plot.width = 7, repr.plot.height = 5, repr.plot.res = 200)
pie_dat = data.frame(value=c(17,110,1147,1368,478,76,2),quality=as.factor(3:9))
ggplot(pie_dat, aes(x = "", y = value, fill = quality)) +
  geom_bar(width = 1, stat = "identity") +
  coord_polar("y", start=0) +theme_void()
```


    
![png](./.github/output_11_0.png?sanitize=true)
    


Từ biểu đồ phân bố về chất lượng của rượu được đánh giá từ 3 đến 9 trên thang điểm 10. Tuy nhiên, thang điểm 5 và 6 chiếm nhiều nhất. Ít nhất là điểm 9 chỉ có 2 loại rượu được đánh giá là 9.

### 3.2. Quan hệ giữa các biến với nhau

Dưới đây là biểu đồ biểu diễn ma trận tương quan giữa các biến dữ liệu trong tập dữ liệu phân tích


```R
options(repr.plot.width = 12, repr.plot.height = 12, repr.plot.res = 250)
chart.Correlation(wine_dat[,1:12],histogram=TRUE,pch="+")
```


    
![png](./.github/output_14_0.png?sanitize=true)
    


<p>Biểu đồ trên cho thấy một số biến có ảnh hưởng đến nhau như sau:

<div align="justify">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Độ chua cố định của rượu <i>(fixed.acidity)</i> bị ảnh hưởng bởi các yếu tố mức độ bay hơi <i>(volatile.acidity)</i>, nồng độ Clo <i>(chlorides)</i>, nồng độ các gốc SO4 <i>(sulphates)</i>, nồng độ axit citric trong rượu <i>(citric.acid)</i> và mức độ đậm đặc của rượu <i>(density)</i> với mức độ ảnh hưởng tăng dần theo tương quan thuận. Trong đó, hai thông số có ảnh hưởng lớn nhất đến độ chua là mức độ đậm đặc của rượu và nồng độ axit citric trong rượu với hệ số tương quan lần lượt là <i>0.56 và 0.41</i>, theo kết quả cho thấy sự khác biệt về độ chua của rượu trong dữ liệu có thể giải thích được <i>16.81%</i> dựa trên sự khác biệt về nồng độ axit citrid của các loại rượu và <i>31.36%</i> dựa trên sự khác biệt về nồng độ. Ngoài ra, độ chua cố định có tương quan nghịch với ... </div>
<div align="justify"></div>

<div align="justify">Tương tự với mức độ bay hơi, độ đậm đặc của rượu và chất lượng quality</div>
</p> 

### 3.3. Phân tích thành phần chính (PCA)


```R
options(repr.plot.width = 7, repr.plot.height = 5, repr.plot.res = 200)
res.pca <- PCA(wine_dat
               ,quanti.sup=12
               ,quali.sup=13
              ,scale.unit=TRUE)
round(res.pca$eig,2)
```


<table class="dataframe">
<caption>A matrix: 11 × 3 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>eigenvalue</th><th scope=col>percentage of variance</th><th scope=col>cumulative percentage of variance</th></tr>
</thead>
<tbody>
	<tr><th scope=row>comp 1</th><td>3.34</td><td>30.40</td><td> 30.40</td></tr>
	<tr><th scope=row>comp 2</th><td>2.36</td><td>21.43</td><td> 51.83</td></tr>
	<tr><th scope=row>comp 3</th><td>1.64</td><td>14.93</td><td> 66.75</td></tr>
	<tr><th scope=row>comp 4</th><td>0.92</td><td> 8.37</td><td> 75.12</td></tr>
	<tr><th scope=row>comp 5</th><td>0.74</td><td> 6.70</td><td> 81.83</td></tr>
	<tr><th scope=row>comp 6</th><td>0.57</td><td> 5.15</td><td> 86.97</td></tr>
	<tr><th scope=row>comp 7</th><td>0.48</td><td> 4.35</td><td> 91.32</td></tr>
	<tr><th scope=row>comp 8</th><td>0.45</td><td> 4.13</td><td> 95.45</td></tr>
	<tr><th scope=row>comp 9</th><td>0.28</td><td> 2.55</td><td> 98.00</td></tr>
	<tr><th scope=row>comp 10</th><td>0.18</td><td> 1.64</td><td> 99.64</td></tr>
	<tr><th scope=row>comp 11</th><td>0.04</td><td> 0.36</td><td>100.00</td></tr>
</tbody>
</table>




```R
options(repr.plot.width = 7, repr.plot.height = 5, repr.plot.res = 200)
barplot(res.pca$eig[,1],main="Eigenvalues",
names.arg=paste("dim",1:nrow(res.pca$eig)))
```


    
![png](./.github/output_18_0.png?sanitize=true)
    


Kết quả cho thấy hai chiều không gian chính Dim1 và Dim2 có thể giải thích được 51.83% các giao động của dữ liệu chưa bao gồm các biến giải thích *quality* và *type*, tỉ lệ giải thích là tương đối thấp tuy nhiên nó có thể diễn giải được cấu trúc cần thiết của dữ liệu, vì phân tích PCA trên gồm có 11 biến dữ liệu và lượng mẫu tương đối rất lớn gần 3200 mẫu thì tỉ lệ giải thích được cần thiết là trên 28.9%. Tuy nhiên sự biến đổi của các yếu tố lý hóa tác động đến chất lượng của rượu không thể chỉ giải thích được dữa trên hai chiều chính.


```R
res.pca$quanti.sup
```


<dl>
	<dt>$coord</dt>
		<dd><table class="dataframe">
<caption>A matrix: 1 × 5 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>Dim.1</th><th scope=col>Dim.2</th><th scope=col>Dim.3</th><th scope=col>Dim.4</th><th scope=col>Dim.5</th></tr>
</thead>
<tbody>
	<tr><th scope=row>quality</th><td>-0.1588748</td><td>-0.1659027</td><td>-0.3776203</td><td>0.1430706</td><td>0.2174304</td></tr>
</tbody>
</table>
</dd>
	<dt>$cor</dt>
		<dd><table class="dataframe">
<caption>A matrix: 1 × 5 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>Dim.1</th><th scope=col>Dim.2</th><th scope=col>Dim.3</th><th scope=col>Dim.4</th><th scope=col>Dim.5</th></tr>
</thead>
<tbody>
	<tr><th scope=row>quality</th><td>-0.1588748</td><td>-0.1659027</td><td>-0.3776203</td><td>0.1430706</td><td>0.2174304</td></tr>
</tbody>
</table>
</dd>
	<dt>$cos2</dt>
		<dd><table class="dataframe">
<caption>A matrix: 1 × 5 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>Dim.1</th><th scope=col>Dim.2</th><th scope=col>Dim.3</th><th scope=col>Dim.4</th><th scope=col>Dim.5</th></tr>
</thead>
<tbody>
	<tr><th scope=row>quality</th><td>0.0252412</td><td>0.0275237</td><td>0.1425971</td><td>0.02046921</td><td>0.047276</td></tr>
</tbody>
</table>
</dd>
</dl>



Qua kết quả đánh giá biến giải thích đối với các chiều không gian cho thấy rằng chất lượng của rượu nó đóng góp tốt nhất ở chiều không gian thứ 3 và thứ 5. Nên ta sẽ chọn chiều không gian này để biểu diễn theo biến chất lượng của rượu để phân tích.

#### Biểu diễn chiều Dim3 và Dim5 để xem xét chất lượng của rượu


```R
options(repr.plot.width = 7, rep.plot.height = 7, repr.plot.res = 250)
plot(res.pca,choix="ind",habillage=12,cex=0.7,axes=c(3,5))
plot(res.pca,choix="var",axes=c(3,5))
```


    
![png](./.github/output_23_0.png?sanitize=true)
    



    
![png](./.github/output_23_1.png?sanitize=true)
    


Nhận xét về các cá thể ta thấy rằng cấu trúc đám mây dữ liệu trong hai chiều này phân chia ra thành cụm chính rõ ràng theo 4 góc phần tư, như hình bốn cánh hoa, ngoài ra ta thấy rằng các điểm dữ liệu vùng biên, hay vùng ngoại lai đa phần là các điểm dữ liệu màu xanh dương hoặc xanh tím, là mức chất lượng thấp nhất trong tập dữ liệu khoảng 3 ~ 4 điểm. Mức độ chất lượng của sản phẩm rượu cũng tăng dần vào tâm của mỗi cánh hoa vì càng vào tâm các cánh hoa ta thấy rằng màu sắc nó chuyển từ tím sáng đỏ và đậm dần, điều này chứng tỏ mức độ điểm chất lượng tăng dần.  

Ngoài ra, ta có thể thấy được rằng chất lượng rượu nó tương quan rất tốt đối với các yếu tố, nồng độ cồn, độ chua cố định, thành phần axit citric, các gốc sulphat, điều này cho thấy chất lượng của rượu có thể tăng thêm nếu các yếu tố này tốt. Tuy nhiên, cũng có những yếu tố khác ảnh hưởng tiêu cực đến chất lượng của rượu như là mức độ bay hơi của rượu, nồng độ clo, độ đậm đặc của rượu.


Tuy nhiên, mức độ ý nghĩa của cả hai chiều này chỉ chiểm khoảng 21.63%, tỉ lệ khá thấp. Hay nói cách khác nếu chỉ dựa vào các phân tích thành phần hóa lý thì chưa đủ thông tin để kết luận được loại rượu vang đó có tốt hay không.


```R
res.pca$quali.sup
```


<dl>
	<dt>$coord</dt>
		<dd><table class="dataframe">
<caption>A matrix: 2 × 5 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>Dim.1</th><th scope=col>Dim.2</th><th scope=col>Dim.3</th><th scope=col>Dim.4</th><th scope=col>Dim.5</th></tr>
</thead>
<tbody>
	<tr><th scope=row>red</th><td> 1.623442</td><td> 0.0494572</td><td> 0.1039025</td><td>-0.003756273</td><td> 0.07915237</td></tr>
	<tr><th scope=row>white</th><td>-1.623442</td><td>-0.0494572</td><td>-0.1039025</td><td> 0.003756273</td><td>-0.07915237</td></tr>
</tbody>
</table>
</dd>
	<dt>$cos2</dt>
		<dd><table class="dataframe">
<caption>A matrix: 2 × 5 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>Dim.1</th><th scope=col>Dim.2</th><th scope=col>Dim.3</th><th scope=col>Dim.4</th><th scope=col>Dim.5</th></tr>
</thead>
<tbody>
	<tr><th scope=row>red</th><td>0.9875037</td><td>0.0009164822</td><td>0.004044986</td><td>5.286633e-06</td><td>0.002347431</td></tr>
	<tr><th scope=row>white</th><td>0.9875037</td><td>0.0009164822</td><td>0.004044986</td><td>5.286633e-06</td><td>0.002347431</td></tr>
</tbody>
</table>
</dd>
	<dt>$v.test</dt>
		<dd><table class="dataframe">
<caption>A matrix: 2 × 5 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>Dim.1</th><th scope=col>Dim.2</th><th scope=col>Dim.3</th><th scope=col>Dim.4</th><th scope=col>Dim.5</th></tr>
</thead>
<tbody>
	<tr><th scope=row>red</th><td> 50.19962</td><td> 1.821258</td><td> 4.584977</td><td>-0.2213375</td><td> 5.212995</td></tr>
	<tr><th scope=row>white</th><td>-50.19962</td><td>-1.821258</td><td>-4.584977</td><td> 0.2213375</td><td>-5.212995</td></tr>
</tbody>
</table>
	<dt>$eta2</dt>
		<dd><table class="dataframe">
<caption>A matrix: 1 × 5 of type dbl</caption>
<thead>
	<tr><th></th><th scope=col>Dim.1</th><th scope=col>Dim.2</th><th scope=col>Dim.3</th><th scope=col>Dim.4</th><th scope=col>Dim.5</th></tr>
</thead>
<tbody>
	<tr><th scope=row>type</th><td>0.7882397</td><td>0.001037529</td><td>0.006575544</td><td>1.532383e-05</td><td>0.008500255</td></tr>
</tbody>
</table>
</dd>
</dl>



Từ kết quả biểu diễn đóng góp của biến giải thích phân loại rượu ta thấy đóng góp rất tốt trong chiều không gian Dim1, và hệ số cos2 của biến gần bằng 1 trong chiều không gian này. Ta biểu diễn nó trên hai chiều không gian chính để quan sát.


```R
options(repr.plot.width = 6, rep.plot.height = 5, repr.plot.res = 250)
plot(res.pca,choix="ind",habillage=13,cex=0.7)
```


    
![png](./.github/output_27_0.png?sanitize=true)
    


Khi biểu diễn dữ liệu theo biến giải thích về phân loại, ta thấy được rõ ràng dữ liệu tách hẳn ra thành 2 cụm dữ liệu riêng biệt là rượu vang đỏ và rượu vang trắng. Như vậy, ta có thể dự đoán rằng theo cách nhìn của hai chiều không gian chính thì rượu vang đỏ và rượu vang trắng sẽ có một vài thành phần hóa lý sẽ hoàn toàn khác biệt với nhau. Hay nói cách khác ta có thể hoàn toàn dựa vào thành phần hóa lý của rượu để dự đoán được loại rượu là vang đỏ hay vang trắng.

## 4. Dự đoán loại rượu dựa trên thành phần chính sau khi phân tích PCA

### 4.1. Xây dựng mô hình máy học

<p align="center"><img src="https://raw.githubusercontent.com/tquangsdh20/winedataset/master/.github/model.svg"></p>

Tiến hành extract dữ liệu để xây dựng model


```R
mydat = data.frame(cbind(res.pca$ind$coord,type=wine_dat$type))
mydat[mydat$type == 2,'type'] <- 'white'
mydat[mydat$type == 1,'type'] <- 'red'
mydat$type <- as.factor(mydat$type)
head(mydat)
```


<table class="dataframe">
<caption>A data.frame: 6 × 6</caption>
<thead>
	<tr><th></th><th scope=col>Dim.1</th><th scope=col>Dim.2</th><th scope=col>Dim.3</th><th scope=col>Dim.4</th><th scope=col>Dim.5</th><th scope=col>type</th></tr>
	<tr><th></th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;fct&gt;</th></tr>
</thead>
<tbody>
	<tr><th scope=row>1</th><td>-1.1583155</td><td> 2.50205297</td><td> 0.2217197</td><td> 1.26054836</td><td> 0.03139377</td><td>white</td></tr>
	<tr><th scope=row>2</th><td>-0.8394453</td><td>-1.23367395</td><td>-1.4868028</td><td>-0.59027497</td><td>-0.70410815</td><td>white</td></tr>
	<tr><th scope=row>3</th><td>-0.8165065</td><td>-0.02031042</td><td> 1.2824748</td><td>-1.41618571</td><td>-0.03556543</td><td>white</td></tr>
	<tr><th scope=row>4</th><td>-3.3597111</td><td> 0.70299773</td><td>-0.5510811</td><td> 0.01412896</td><td>-0.26905030</td><td>white</td></tr>
	<tr><th scope=row>5</th><td>-2.1528084</td><td> 0.60018393</td><td>-0.5024772</td><td>-0.64938786</td><td> 0.48874428</td><td>white</td></tr>
	<tr><th scope=row>6</th><td>-2.8004525</td><td> 2.84244601</td><td> 0.3240747</td><td>-0.16091926</td><td> 0.27056494</td><td>white</td></tr>
</tbody>
</table>



Tách biệt hai tập dữ liệu để training và testing cho model máy học


```R
set.seed(1)
dat_smpl = sample.split(mydat$type,SplitRat=0.8)
train_dat = data.frame(subset(mydat,dat_smpl == T))
test_dat = data.frame(subset(mydat,dat_smpl == F))
train_dat$type <- as.factor(train_dat$type)
test_dat$type <- as.factor(test_dat$type)
summary(train_dat$type)
```


Xây dựng CART Model


```R
CART_model = rpart(type ~ ., train_dat)
options(repr.plot.width = 6, rep.plot.height = 6, repr.plot.res = 200)
rpart.plot(CART_model, extra = 0, type=5, tweak=1.2)
```


    
![png](./.github/output_34_0.png?sanitize=true)
    


## 4.2. Đánh giá model


```R
# Predict fraud classe with SMOTE sampling
print("The test data")
summary(test_dat$type)
print("Prediction")
predicted = predict(CART_model, test_dat, type = 'class')
summary(predicted)
```

    [1] "The test data - red : 320 - white : 320"
    [2] "Prediction - red : 319 - white : 321"
    



Từ trên cho thấy kết quả thực tế của tập dữ liệu testing thì có 320 trường hợp là rượu vang đỏ và 320 trường hợp là rượu vang trắng. Sau đó sẽ đem đi tiến hành dự đoán kết quả.


```R
confusionMatrix(predicted,test_dat$type)
```


    Confusion Matrix and Statistics
    
              Reference
    Prediction red white
         red   314     5
         white   6   315
                                              
                   Accuracy : 0.9828          
                     95% CI : (0.9695, 0.9914)
        No Information Rate : 0.5             
        P-Value [Acc > NIR] : <2e-16          
                                              
                      Kappa : 0.9656          
                                              
     Mcnemar's Test P-Value : 1               
                                              
                Sensitivity : 0.9812          
                Specificity : 0.9844          
             Pos Pred Value : 0.9843          
             Neg Pred Value : 0.9813          
                 Prevalence : 0.5000          
             Detection Rate : 0.4906          
       Detection Prevalence : 0.4984          
          Balanced Accuracy : 0.9828          
                                              
           'Positive' Class : red             
                                              


Theo như kết quả dự đoán thì ta thấy rằng model máy học trên dự đoán đúng 314/320 trường hợp rượu vang đỏ dựa trên thành phần lý hóa và dự đoán đúng được 315/320 trường hợp rượu vang trắng. Tỷ lệ chính xác của model theo kết quả trên là 98.28%.

## 5. Kết luận

Như vậy qua kết quả phân tích và đánh giá ta có thể rút ra được những kết luận sau đây:

- Không thể đánh giá chất lượng của rượu vang chỉ dựa trên các thành phần hóa lý. Trên thực tế nguyên liệu, các thành phần hóa lý của rượu chỉ đóng góp một phần trong việc làm tăng hương vị của rượu, ngoài ra điểm quan trong nữa để khiến rượu ngon có thể là do phương pháp ủ rượu và môi trường ủ rượu... hay các yếu tố mà không thể thống kê được

- Đối với việc phân loại rượu vang thì ngược lại, dựa vào kết quả phân tích chính của các yếu tố hóa lý của nó ta hoàn toàn có thể phân loại nó bằng cách ứng dụng các model máy học, để xây dựng một ứng dụng dự đoán loại rượu dựa trên các thành phần lý hóa của nó.
