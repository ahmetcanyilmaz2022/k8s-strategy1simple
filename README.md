# k8s-strategy1simple
RECREATE VS ROLİNG UPDATE
DevopsThunder
DevopsThunder

4 min read
·
Just now




Selamlar

2 bölümden oluşan bu yazı dizimizin birinci ayağında Kubernetes Deployment Stratejileri hakkında bilgiler vereceğiz ve küçük örneklemelerle bitireceğiz .

Deployment Stratejilerini ne için kullanırız sorusunun yanıtını sesli düşüncemi yazıya dökerek kısaca özetleyim çalışmakta olan bir V1 uygulamamızın olduğunu var sayalım ve zamanla kullanıcı ve iş gereksinimleri bu dürümü V2 yapmamızı gerektiriyor . İşte tam burada Deployment stratejileri devreye giriyor .

Duraksama olmadan nasıl güncellerim / en hızlı bu işi hangi yöntemle başarırım gibi soruların cevabıbını bu yazı dizimizde netliğe kavuşturacağız .

RECREATE

Basit ve kafamdaki düşüncelerden örnek vereyim ve şablona dökelim ufakta bir uygulama yapalım .

3 Pod dan oluşan bir deployment Objemiz Prod veya Test ortamında bir uygulama çalıştırır ve versiyon V1 diyelim .

zamanla müşteri ve piyasa ihtiyaçlarına yönelik ihtiyaçlar doğrultusunda Geliştiriciler tarafından uygulamada çalışan imaj yapısı güçlendirildi güncellendi ve daha release hale geldi bunun da adnı V2 koydular :)

Biz Birazdan aşağıda yapacağım uygulama ve vereceğim komutlarda daha iyi göreceğiz ve yapacağımız bir takım işlerle v2 versiyona güncelledik harika uygulama çalısıyor

Fakat ; Recreate de şöyle bir yapılanma durumu mevcut deployment içerisindeki Tüm aynı anda podlar Terminate olur ve Terminate sonrası yeni v2 podlar pending aşamasından Run durumuna geçer ..

Bu terminate ile Runing arasında bir kesintiye maruz kalmamız muhtemel

Bu yüzden Recreate durumunu kendi öngörüm olarak sadece test ortamlarında kullanmamız önerimdir .

RECREATE UYGULAMA
Aşağıdaki yaml dosyamızda bir Deployment yaratacağız . incelediğiniz üzere spec altında strategy ve type bölünü göreceksiniz . Bu kısımdan stratejimizi beliliyoruz

Google ın yaratmış olduğu hello-app:1.0

imajı ile 4 replikadan oluşan bir deployment oluşturuyoruz

apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 4
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080

kubectl apply -f nameofthe.yaml
Şimdi Bir büyüğümüz bize dediki; hello-app:1.0 uygulamamızı hello-app:2.0 yaparmısın :)

Hemen Deployment a ait yaml dosyamıza erişiyoruz veya kubectl edit komutu ile de ilerleyebiliriz . image kısmı düzenlemelerimizi 2.0 versiyon olarak güncelliyoruz .

apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 4
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello
        image: gcr.io/google-samples/hello-app:2.0
        ports:
        - containerPort: 8080
ve ne diyoruz :)

kubectl apply -f dosyaadı.yaml
Not : İzlemeniz adına tavsiyem Kullanmış olduğunuz editör veya bulut sağlayıcısı Shellinde 2. bir ekranı açıp <kubectl get pods -w> diyerek potlar arası terminate ve runing durumlarını canlı izleyebilirsiniz.


Toplu imha
Böylece Recreate stratejisinde ki olayı yukardaki görselden de anladığımız üzere versiyon güncellenecek çalışan podlara önce toplu imha ve sonra yeniden oluşturmaya şahit olduk .

ROLLING UPDATE

Kubernetes üzerindeki Default Strateji yöntemidir.

Bu yöntemde çok basit bir anlatımda Deployment içerisindeki podlar toplu olarak versiyon değişikliğine gitmez .. Deployment içerisinde 4 replika yani pod olduğunu düşünelim . Yukardaki resimde anlaşıldığı üzere 1. v1 pod teminate olmasıyla , 1. v2 pod runing olmaya başlar ardından 2 , 3, 4. pod şeklinde ilerler böylece sistemde kesinti ve aralık yaşamayız .

ROLLING UPDATE UYGULAMA
aşağıda yer alan yaml dosyalarını inceleyelim ve özellikle spec altında yer alan strategy ve type kısmını iyi özümseyelim .

apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080

maxSurge: 1 ve maxUnavailable:0
maxSurge: 1 ve maxUnavailable:0 ===> herseferinde 1 pod terminate et ve bir pod ayağa kaldır isteğidir bu tarafta değişiklik özgün ve çalıştığınız yapıya göre farklılığa gidiebilir.

kubectl apply -f rollingupdate.yaml

// rolling update stratejisi barındıran yukarda bilgileri olan
// rollingupdate.yaml dosyamızla hello-app:1.0 imajlı deployment objemizi 
// ayağa kaldıralım

tekrardan :) Bir büyüğümüz bize dediki; hello-app:1.0 uygulamamızı hello-app:2.0 yaparmısın :)

Hemen Deployment a ait yaml dosyamıza erişiyoruz veya kubectl edit komutu ile de ilerleyebiliriz . image kısmı düzenlemelerimizi 2.0 versiyon olarak güncelliyoruz .

apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello
        image: gcr.io/google-samples/hello-app:2.0  ##>>>>>>>>buraya dikkat
        ports:
        - containerPort: 8080
kubectl apply -f rolling-update.yaml 

lütfen 2. ekrandan <kubectl get pods -w>  yapalım canlı olarak izleyelim:) 

aşağıdaki görselde anlaşıldığı üzere çalısan uygulamada toplu bir kesintisi olmadan sırasıyla runing ve terminate işlemleri gerçekleşti.
