# Term-Project-2: ICIP 登革熱
Data來自Aldea人工智慧共創平台的"尋找病媒蚊孳生源-積水容器影像物件辨識"競賽。(https://aidea-web.tw/topic/47b8aaa7-f0fc-4fee-af28-e0f077b856ae) 目的是藉由機器學習，偵測照片中的積水容器。並且期盼能透過影像物件偵測技術，讓稽查人員藉由影像或是視訊提醒其積水容器之物件位置，提高稽查效率。並且期望能更進一步運用於其他載具中，協助登革熱的防治。

而這次，所需要辨識的物件種類有aquarium, bottle, bowl, box, bucket, plastic_bag, plate, Styrofoam, tire, toilet, tub, washing_machine, water_tower等13類。

所提供的training data檔案有兩個類型.jpg和.xml。.jpg檔為欲偵測的照片，.xml檔則是照片中所有前述13類物件的類型、bounding box的長寬以及bounding box的位置。

## Data Processing & Analyze
由於所提供的檔案格式不利於分析，所以先將.xml檔匯集成一份csv檔，如下圖。
![](https://imgur.com/FPVq3DR.png)

filename:照片檔案名稱

width:bounding box寬

height:bounding box高

class:物件種類

xmin:bounding box左上X座標

ymin:bounding box左上y座標

xmax:bounding box右下x座標

ymax:bounding box右下y座標

### Data analyze
針對training data 2671張照片中中所有類型的物件數量做簡單的統計，得到的結果如下圖，可以看出分布不是很平均。
![](https://imgur.com/t1hCu6Y.png)



## Training
對於該使用哪種方法來進行物件偵測的訓練，我們找了yolo與faster rcnn這兩個方式來比較，而yolo2對小物體預測效果似乎較差，而且其他組別基本上都是用yolo。所以我們就決定嘗試faster rcnn，並且在Tensorflow的架構下進行物件偵測的訓練。

### Further processing
由於我們選定了在Tensorflow的架構下進行物件偵測的訓練，因此我們又將.jpg與前面所整合的csv檔合併成一個binary的檔案。而這個檔案為TFRecord格式，專門在Tensorflow的架構下使用。而由於這個檔案有700+mb所以也就不附上，

而為了在Tensorflow的架構下進行訓練，我們也產生.pbtxt檔案來定義calss的代號與名稱。檔案內容如下圖
![](https://imgur.com/38QYmnI.png)

### Pre-train model
在進一步研究與嘗試自行訓練行後，發現主辦所提供的data數量不足以從頭訓練自己的模型。再加上，電腦效能不夠好，從頭自行訓練模型可能無法在時限內完成成果。所以最後我們選擇利用其他Pre-train model的參數，再針對第三層neural layer進行訓練，以縮短訓練所需的時間。

而在這裡，我們選擇了用faster rcnn來針對微軟的COCO dataset訓練的model(faster_rcnn_inception_v2_coco)。訓練時的loss如下圖，約在10000個step後趨緩。也因此我們就在1萬多個step停止訓練。
![](https://imgur.com/TK4ErZK.png)

## Result
訓練完將frozen inference graph輸出後，就可以針對test data進行predict，並同時將predict的結果輸出成上傳用的格式，如下圖。

![](https://imgur.com/COcqy9y.png)
Image_filename: 檔名

label_id: 預測種類(1～13)

x: x min，預測方框的左上x座標

y: y min，預測方框的左上y座標

w: 預測方框的寬度(x軸)

h: 預測方框的高度(y軸)

Confidence:預測物件 bounding box 之信心水準

而遇刺結果的視覺化呈現如下。

![](https://imgur.com/TBRxxRC.png)

上傳後的成績如下。

![](https://imgur.com/GtxEigh.png)

## Summary & further trail
由於前面的成績看起來不是很理想，因此，我們想了幾個原因：1.faster rcnn不適合。2.threshold設定太高導致部分有被測到但confidence不夠高的預測沒被submit。3.train data不足導致訓練出來的model不佳。接下來我們就針對這三點進行修正並重新訓練，同樣都進行約1萬多個step以利比較

### Different method(trail 2)
在這裡我們選擇用SSD演算法，並同樣是以微軟coco dataset訓練的model(ssd_mobilenet_v1_coco_2017_11_17)進行訓練。
上傳後的成績如下。

![](https://imgur.com/wLcBS41.png)

可以看到成績變差了(submission2)，所以之後我們又改回前面所使用的pre-train model。

### Lower confidence threshold(trail 3)
這裡所使用的是第一次所訓練好的model，然後將confidence的threshold降到0.2。得到的成績如下。

![](https://imgur.com/GAocA8Z.png)
可以看到成績比起前兩個顯著較高(submission3)。而在這裡，算是證實的我們的猜測，也就是說，物件有被找出來，但是confidence不知為何搞砸了。

### Data augmentation(trail 4)
我們採用改變色調的方式進行資料增強，如下圖，將資料擴充成四倍並重新產生TFRecord檔。

![](https://imgur.com/xJm3mdG.png)

訓練完預測的結果如下。可以看到結果並沒有比最初的結果好。(submission4)

![](https://imgur.com/awTMZue.png)

## Final result
Private Leaderboard與排名如下圖，這裡會用submission4而不是最高的submission3是因為我疏忽了，忘記後面submit的結果會蓋掉前面的。

![](https://imgur.com/75zsmJh.png)

## Discussion
由於最後時間不夠再進行更多的修正與嘗試，所以我們就停在這裡了。而我們針對這次的競賽覺得在未來還能試試看從頭train自己的model，因為利用Pre-train model進行訓練時，基本上是只針對最後一層的neural layer進行訓練，而這樣的話可以微調的方向就不多。

此外，嘗試YOLO演算法也是相當值得試試看的，因為在做演算法的調查時，其實可以看到YOLO演算法無論是在所花的時間，以及準確度的表現都是最突出的。

再來，擴大dataset也是相當必要的，因為不論是要從頭train自己的model或是要改善前面分析所看到分布不均的情形，都會需要擴大dataset。

最後就是針對這種deep learning的原理與進行方式，我們還得再更加深入的了解，這樣才有辦法做到比較多面向，且比較準確的tuning。

