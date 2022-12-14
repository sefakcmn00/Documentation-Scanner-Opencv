# Documentation-Scanner-Opencv
-----------------------------------------------
It is the documentation scanner project of our Computer Vision work with Opencv. In this project, documents shown to the computer camera were scanned using Opencv artificial intelligence libraries. Below I tried to explain the details of the project step by step.


## Documantation-Sacnner-Opencv

First, we will download the libraries we will use.
```Python
import cv2
import numpy as np
import utlis

```

### Camera Options

```Python
webCamFeed = True
cap = cv2.VideoCapture(1)
cap.set(10, 160)
heightImg = 640
widthImg = 480
```
We add the necessary lines of code to start our webcam in the operations here.
If you do not want to operate with the webcam, you can also scan any document image into the program. (pathImage = "image name".
We have defined #camera screen widths above.
```Python
utlis.initializeTrackbars()
count = 0
```
```Python
while True:

    if webCamFeed:
        success, img = cap.read()
    else:
        img = cv2.imread(pathImage)
    img = cv2.resize(img, (widthImg, heightImg))  #GÖRÜNTÜYÜ YENİDEN BOYUTLANDIRMA AMACIYLA EKLEDİĞİMİZ KOD.
    imgBlank = np.zeros((heightImg, widthImg, 3), np.uint8)  # GEREKİRSE HATA AYIKLAMANIN TEST EDİLMESİ İÇİN BOŞ BİR GÖRÜNTÜ OLUŞTURMAK AMACIYLA EKLEDİĞİMİZ KOD.
    imgGray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)  # GÖRÜNTÜYÜ GRİ ÖLÇEKLE DÖNÜŞTÜRMEK AMACIYLA GİRİLEN KOD.
    imgBlur = cv2.GaussianBlur(imgGray, (5, 5), 1)  # GAUSS BLUR EKLEMEK AMACIYLA GİRİLEN KOD.
    thres = utlis.valTrackbars()  # EŞİKLER İÇİN İZLEME ÇUBUĞU DEĞERLERİNİ ALMAK AMACIYLA GİRİLEN KOD.
    imgThreshold = cv2.Canny(imgBlur, thres[0], thres[1])  # CANNY BLUR UYGULAYIN
    kernel = np.ones((5, 5))
    imgDial = cv2.dilate(imgThreshold, kernel, iterations=2)  # GENİŞLETME UYGULAMASI.
    imgThreshold = cv2.erode(imgDial, kernel, iterations=1)  # EROZYON UYGULAMASI.

```
```Python
####################################  CONTOUR İŞLEMLERİ  #################################################################
    ## TÜM CONTOUR'LARI BULMAK AMACIYLA AŞAĞIDAKİ KOD SATIRLARINI EKLİYORUZ.
    ###
    #CONTOUR Nedir? Konturlar, aynı renk veya yoğunluğa sahip tüm sürekli noktaları (sınır boyunca) birleştiren bir eğri olarak basitçe açıklanabilir.
    #Konturlar, şekil analizi ve nesne algılama ve tanıma için kullanışlı bir araçtır.
    ###
    imgContours = img.copy()  # GÖRÜNTÜ AMAÇLI KOPYALAMA
    imgBigContour = img.copy()  # GÖRÜNTÜ AMAÇLI KOPYALAMA
    contours, hierarchy = cv2.findContours(imgThreshold, cv2.RETR_EXTERNAL,
                                           cv2.CHAIN_APPROX_SIMPLE)  # TÜM KONTURLARI BUL
    cv2.drawContours(imgContours, contours, -1, (0, 255, 0), 10)  # TESPİT EDİLEN TÜM KONTURLARI ÇİZİM
```
```Python
 # EN BÜYÜK COUNTOUR'U BULMA
    biggest, maxArea = utlis.biggestContour(contours)
    if biggest.size != 0:
        biggest = utlis.reorder(biggest)
        cv2.drawContours(imgBigContour, biggest, -1, (0, 255, 0), 20)  # EN BÜYÜK KONTUR
        imgBigContour = utlis.drawRectangle(imgBigContour, biggest, 2)
        pts1 = np.float32(biggest)  # ÇÖZGÜ İÇİN NOKTALARI HAZIRLA
        pts2 = np.float32([[0, 0], [widthImg, 0], [0, heightImg], [widthImg, heightImg]])  # ÇÖZGÜ İÇİN NOKTALARI HAZIRLA
        matrix = cv2.getPerspectiveTransform(pts1, pts2)
        imgWarpColored = cv2.warpPerspective(img, matrix, (widthImg, heightImg)) #warp perspektif yani görünütü çarpıtma işlemleri

        # HER TARAFTAN 20 PİKSEL FORMUNU KALDIR
        imgWarpColored = imgWarpColored[20:imgWarpColored.shape[0] - 20, 20:imgWarpColored.shape[1] - 20]
        imgWarpColored = cv2.resize(imgWarpColored, (widthImg, heightImg))

        # UYARLANABİLİR EŞİK UYGULAMA
        imgWarpGray = cv2.cvtColor(imgWarpColored, cv2.COLOR_BGR2GRAY)
        imgAdaptiveThre = cv2.adaptiveThreshold(imgWarpGray, 255, 1, 1, 7, 2)
        imgAdaptiveThre = cv2.bitwise_not(imgAdaptiveThre)
        imgAdaptiveThre = cv2.medianBlur(imgAdaptiveThre, 3)

        # Görüntüleme için Görüntü Dizisi
        imageArray = ([img, imgGray, imgThreshold, imgContours],
                      [imgBigContour, imgWarpColored, imgWarpGray, imgAdaptiveThre])

    else:
        imageArray = ([img, imgGray, imgThreshold, imgContours],
                      [imgBlank, imgBlank, imgBlank, imgBlank])

    # EKRAN ETİKETLERİ
    lables = [["Orijinal", "Gri", "Esik", "Contours"],
              ["En buyuk Contour", "Warp Perspektif", "Warp Gri", "Uyarlanabilir Esik"]]

    stackedImage = utlis.stackImages(imageArray, 0.75, lables)
    cv2.imshow("Program", stackedImage)
```
# Program Output

![myimage0](https://user-images.githubusercontent.com/67556543/182640541-1ebc36e9-c126-4195-9740-50f52b355301.jpg)

![myimage2](https://user-images.githubusercontent.com/67556543/182640564-73061e56-0853-407d-8981-de62a9e7f996.jpg)
