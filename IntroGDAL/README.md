# GDAL(Geospatial Data Abstraction Library)

 "Open Source Geospatial Foundation" tarafından "X/MIT" stili Açık Kaynak lisansı altında yayınlanan, raster ve vektör coğrafi veri biçimleri için bir çevirmen kitaplığıdır. <br>
 Farklı dosya biçimlerinden, veri türlerinden ve harita projeksiyonlarından gelen verilerin çevrilmesine yardımcı olan bir yazılım koleksiyonudur.

## CLI (Command Line Interface)

### GDAL Versiyonu: <br>
gdalinfo --versiyon <br>

### Görüntü Verilerini Keşfetmek: <br>
Veri kümesi hakkında temel bilgileri almak için gdalinfo'yu kullanırız.

gdalinfo dataset.png <br>
gdalinfo -mm east.dem <br>
(-mm anahtarı nedeniyle minimum/maks yükseklik değerleri gösterilir.)

### Sürücüleri Keşfetmek: <br>
Formatlar <br>
gdalinfo --formats  <br>
(Mevcut biçim sürücülerinin bir listesini görmek için kullanılabilir.)<br>

gdalinfo --format jpeg  <br>
(--format komut satırı anahtarı, oluşturma seçenekleri ve izin verilen veri türleri dahil olmak üzere belirli bir sürücü hakkındaki ayrıntıları sorgulamak için kullanılabilir . )<br>

Her format, eğer varsa <br>
salt okunur (ro),<br>
okuma/yazma (rw) veya<br>
okuma/yazma/güncelleme (rw+).<br>


### Çeviri:<br>
Çeviriler gdal_translate komutu ile yapılır. <br>
gdal_translate -of "output format" "source filename" "destination filename" <br>

"gdal_translate"  <br>
"-of" çıktı biçimini ayarlar. <br>
"-ot" anahtarı çıktı veri türünü değiştirmek için kullanılabilir.  (Örn: gdal_translate -ot Byte east.dem bytedem.tif) <br>
"-scale" anahtarı, verileri yeniden ölçeklendirmek için kullanılabilir.  Giriş ve çıkış aralıklarının açık kontrolü de mevcuttur. (Örn: gdal_translate -of JPEG -scale -100 678 0 255 east.dem east.jpg )<br>



## CPP

### Dosyayı Açma

GDAL destekli bir raster veri deposunu açmadan önce sürücülerin kaydedilmesi gerekir. <br>
Bu "GDALAllRegister()" ile gerçekleşir. Bu işlev genellikle uygulamanın başında bir kez çağrılmalıdır.

Sürücüler kaydedildikten sonra, bir veri kümesi açmak için veri kümesinin adını ve istenen erişimi (GA_ReadOnly veya GA_Update) ileterek "GDALOpen()" işlevi çağırmalıdır.

### Veri Kümesi Bilgilerini Alma

GDAL Veri Kümesi:
Bir veri kümesi, "GDALDataset" sınıfı tarafından temsil edilir ve ilgili tarama bantlarının ve hepsinde ortak olan bazı bilgilerin bir derlemesidir. <br>
Özellikle veri kümesi, tüm bantlar için geçerli olan bir raster boyutu (piksel ve satır olarak) kavramına sahiptir. Veri seti ayrıca tüm bantların coğrafi referans dönüşümü ve koordinat sistemi tanımından sorumludur. <br>


Bir raster bant, GDAL'de "GDALRasterBand" sınıfıyla temsil edilir. Tek bir raster bant/kanal/katmanı temsil eder.

### Raster Verilerini Okuma
Raster verilerini okumanın birkaç yolu vardır, ancak en yaygın olanı "GDALRasterBand::RasterIO()" yöntemidir.

CPLErr GDALRasterBand :: RasterIO ( GDALRWFlag eRWFlag,<br>
                                  int nXOff, int nYOff, int nXSize, int nYSize,<br>
                                  boşluk * pData, int nBufXSize, int nBufYSize,<br>
                                  GDALDataType eBufType,<br>
                                  int nPixelSpace,<br>
                                  int nLineSpace)<br>


eRWFlag (GF_Read veya GF_Write) ayarına göre okumak veya yazmak için aynı RasterIO() çağrısının kullanılır.<br>
nXOff, nYOff, nXSize, nYSize bağımsız değişkeni, diskteki okunacak (veya yazılacak) tarama verilerinin penceresini tanımlar.<br>
pData, verilerin okunduğu veya yazıldığı bellek arabelleğidir. Gerçek türü, GDT_Float32 veya GDT_Byte gibi eBufType olarak iletilen her şey olmalıdır.<br>
nBufXSize ve nBufYSize değerleri arabelleğin boyutunu tanımlar.<br>
nPixelSpace ve nLineSpace normalde sıfırdır ve bu varsayılan değerlerin kullanılması gerektiğini gösterir. Bununla birlikte, örneğin ara piksel eklenmiş diğer verileri içeren bir arabelleğe okumaya izin vererek, bellek veri ara belleğine erişimi kontrol etmek için kullanılabilirler.<br>


### Dosya Oluşturma Teknikleri
Biçim sürücüsü oluşturmayı destekliyorsa, GDAL destekli biçimlerde yeni dosyalar oluşturulabilir. CreateCopy() ve Create() kullanarak dosya oluşturmak için iki genel teknik vardır.<br>
CreateCopy yöntemi, biçim sürücüsünde CreateCopy() yönteminin çağrılmasını ve kopyalanması gereken bir kaynak veri kümesinin iletilmesini içerir. Create yöntemi, sürücüde Create() yönteminin çağrılmasını ve ardından tüm meta verilerin ve tarama verilerinin ayrı çağrılarla açıkça yazılmasını içerir. Yeni dosyalar oluşturmayı destekleyen tüm sürücüler CreateCopy() yöntemini destekler, ancak yalnızca birkaçı Create() yöntemini destekler. <br>

Belirli bir biçimin Create veya CreateCopy'yi destekleyip desteklemediğini belirlemek için biçim sürücüsü nesnesindeki DCAP_CREATE ve DCAP_CREATECOPY meta verilerini kontrol etmek mümkündür. <br>

Create() yöntemi, CreateCopy()'ye çok benzeyen bir seçenekler listesi alır, ancak görüntü boyutu, bant sayısı ve bant türü açıkça sağlanmalıdır. <br>

Veri kümesi başarıyla oluşturulduktan sonra, tüm uygun meta veriler ve tarama verileri dosyaya yazılmalıdır.  (SetGeoTransform,  SetProjection --- >  RasterIO ( GF_Write,...))<br>



### Veri Kümesini Kapatma
GDALDataset'ler "GDALClose()" çağrılarak kapatılabilir.


### Hata?
Tüm GDAL, hata raporlama için CPLError() kullanır.




## Formatlar Hakkında Bilgiler

### GTiff – GeoTIFF Dosya Formatı:  <br>
Görüntüdeki her pikseli Dünya yüzeyinde yerleştirmek için gerekli bilgileri depolayan özel bir TIFF türü.

### JP2OpenJPEG – OpenJPEG kitaplığına dayalı JPEG2000 sürücüsü: <br>
Bu sürücü, OpenJPEG kitaplığı v2'ye dayalı bir JPEG2000 okuyucu/yazıcı uygulamasıdır . JPEG 2000, dalgacık teknolojisine dayalı son teknoloji sıkıştırma tekniklerini kullanan ve son derece yüksek düzeyde ölçeklenebilirlik ve erişilebilirlik sunan bir görüntü kodlama sistemidir. 

### NITF – National Imagery Transmission Format:<br>

### PCIDSK – PCI Geomatics Database File:<br>
Görüntü analizi için PCI EASI/PACE yazılımı tarafından kullanılan PCIDSK veritabanı dosyasıdır.


### HFA – Erdas Imagine .img:<br>
GDAL, okuma erişimi ve yazma için Erdas Imagine .img formatını destekler.

### PNG – Portable Network Graphics:<br>
GDAL, .png dosyalarını okuma ve oluşturma desteği içerir. PNG dosyaları, CreateCopy() yöntemi kullanılarak bir tür PNG ile oluşturulabilir .
XMP meta verileri dosyadan çıkarılabilir ve xml:XMP meta veri etki alanında XML ham içeriği olarak depolanır.

### ECW – Enhanced Compressed Wavelets (.ecw): <br>
GDAL, Hexagon Geospatial (eski adıyla Intergraph, ERDAS, ERMapper) tarafından geliştirilen ERDAS ECW/JP2 SDK kullanılarak ECW dosyalarının okunmasını ve yazılmasını destekler. Destek isteğe bağlıdır ve ECW/JP2 SDK İndirme sayfasında bulunan kitaplıklarda bağlantı kurulmasını gerektirir.<br>
ERDAS ECW/JP2 SDK v5.x, birden çok lisans türü altında mevcuttur. Masaüstü kullanımı için, herhangi bir boyuttaki ECW/JP2 görüntüsünün kodunun çözülmesi ücretsiz olarak sağlanır.
ERDAS ECW/JP2 v4.x ve v5.x SDK yalnızca görüntü açma için ücretsizdir. Görüntüleri sıkıştırmak için okuma/yazma SDK'sı ile derlemek ve çalışma zamanında ERDAS'tan satın alınabilecek bir OEM lisans anahtarı sağlamak gerekir.<br>
Hala ECW 3.3 SDK kullananlar için, 500 MB'tan küçük resimler ücretsiz olarak sıkıştırılabilirken, daha büyük resimler ERDAS'tan lisans alınmasını gerektirir. 

ECW formatında sıkıştırılacak dosyalar da en az 128x128 olmalıdır. ECW şu anda ECW Sürüm 2 dosyaları için kanal başına yalnızca 8 biti desteklemektedir. ECW Sürüm 3 dosyaları, kanal başına 16 biti destekler (Uint16 veri türü olarak).


<!--
JPEG_QUALITY=[1-100] : JPEG sıkıştırmasını kullanırken JPEG kalitesini ayarlayın. 100 değeri en iyi kalitedir (en az sıkıştırma), 1 ise en kötü kalitedir (en iyi sıkıştırma). Varsayılan 75'tir.
-->


## Kaynaklar
* [GDAL API Tutorial](https://gdal.org/tutorials/raster_api_tut.html)
* [GDAL API Tutorial 2](http://pkg.cs.ovgu.de/LNF/i386/5.10/LNFgdal-docs/reloc/gdal/html/gdal_tutorial.html)
* [GDAL Data Model](http://pkg.cs.ovgu.de/LNF/i386/5.10/LNFgdal-docs/reloc/gdal/html/gdal_datamodel.html)
* [GDAL Dataset Class Refence](http://pkg.cs.ovgu.de/LNF/i386/5.10/LNFgdal-docs/reloc/gdal/html/classGDALDataset.html#_details)
* [OGR Projections Tutorial](http://pkg.cs.ovgu.de/LNF/i386/5.10/LNFgdal-docs/reloc/gdal/html/ogr/osr_tutorial.html)
* [PCIDSK Tutorial](https://web.archive.org/web/20130810131727/http://home.gdal.org/projects/pcidsk/tutorial.html)
