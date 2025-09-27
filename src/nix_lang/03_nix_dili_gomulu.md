# Nix Dilinde Gömülü Fonksiyonlar

Önceki makalede Nix dilinin temel yapılarını ve syntax'ını incelemiştik. Bu makalede ise Nix dilinin built-in fonksiyonlarını ve ve çok temel modüllerini inceleyeceğiz.

Bu fonksiyonlar olmadan diyebilir ki sayı toplama çıkartmadan ileriye gidemezdik. Bunu aslında diğer dillerde de gelen temel modüllerle aynı görebilirsiniz.

Nix dilini dil olarak incelediğimizde aslında çok büyük bir dil olmadığını görebiliyoruz. Yani bir çok programlama diline göre az kuralı var. Onu karmaşıkmış gibi yapan sebeplerden biri birçoğumuzun fonksiyonel bir dille daha önce çalışmamış olması ve bir programlama dili olması yanında bir konfigürasyon dili olması.

Builtins fonksiyonlar zaten Nix dilinin bir parçası olduğu için herhangi bir import işlemi yapmamıza gerek yok.

Amacımız halen salt Nix dilinde uzmanlaşmak. Bu yüzden NixOS veya Nixpkgs ile ilgili konulara girmeyeceğiz. Bu makalede daha çok temel fonksiyonları göreceğiz ve mümkün oldukça örnekleri repl üzerinde yapacağız. Bir sonraki makalemizde ise burada gördüğümüz fonksiyonları kullanarak bir modül oluşturup bunu kullanacağız.

Debugging ve Error Handling kavramlarıyla başlayalım.

## Debug ve Error Handling

Nix Dilinde doğrudan debug kavramı veya breakpoint veya IDE'nin desteği ile kodu satır satır çalıştırma ne yazık ki yok. Bunun yerine kod içinde değerleri kontrol edebileceğimiz yardımcı fonksiyonlar mevcut.

### `builtins.trace` fonksiyonu

Birinci birinci bloğu çalıştırıp ekrana basar daha sonra ikinci bloğu çalıştırıp sonucu ekrana basar.

**nix repl** komutu ile repl'i açıp aşağıdaki kodu çalıştırıyoruz.

```nix
builtins.trace (1 + 2) (4 + 5)
# sonuç
# trace: 3
# 9
```

Biraz daha karmaşık bir örnek yapalım.

```bash
d = a:b:a+b
sum = a: b: a + (b (3 5))
mul = a: b: a * b
builtins.trace (sum (2 d)) (mul 4 5)

# sonuç
# trace: <LAMBDA>
# 20
```

Görüleceği üzere **builtins.trace** iç içe fonksiyonlarda veya yapılarda çalışamıyor. Çlk çağrılan fonksiyonun sadece içinde bir lamda recursive yapı görünce bunun bir LAMBDA olduğunu söyleyebildi o kadar.

### `assert` fonksiyonu

Çalışma zamanında bir değerin kontrolu için kullanılır. Eğer beklenen değer gelmezse hata fırlatır.

```bash
assert 2 == 2;
(1 + 2)
assert 2 == 3;
(1 + 2)

# error: assertion '(2 == 3)' failed

#        at «string»:1:1:

#             1| assert 2 == 3;
#              | ^
#             2| (1 + 2)

```

Bu trace'e göre daha kullanışlı bir fonksiyon gibi görünüyor. En azından çalışma zamanında hatalı olması muhtemel değerlerin testinde kullanılabilir.

### `throw fonksiyonu`

Bazen de kendimiz özellikle hata fırlatmak isteyebiliriz. Bu durumda throw fonksiyonunu kullanabiliriz.

```bash

if 1 == 1 then
(1 + 1)
else
throw "1 eşit değil 1 değil"

# sonuç 2

if 1 > 1 then
(1 + 1)
else
throw "1 eşit değil 1 değil"

# sonuç error: 1 eşit değil 1 değil

```

### `builtins.tryEval` fonksiyonu

Hataları susturmak için kullanılır.

```bash

if 1 == 1 then
(1 + 1)
else
builtins.tryEval (throw "1 eşit değil 1 değil")

# result 2


if 1 > 1 then
(1 + 1)
else
builtins.tryEval (throw "1 eşit değil 1 değil")
# result { success = false; value = false; }


builtins.tryEval (1 + 1)
# result { success = true; value = 2; }

```

### `abort` kullanımı

Aslında throw'dan çok da farklı değil. Sadece throw'dan farklı olarak çalışmanın bir hatadan dolayı sonlandırıldığını belirtir.

```bash

abort "1 eşit değil 1 değil"
# result  error: evaluation aborted with the following error message: '1 eşit değil 1 değil'

```

## Temel Liste Fonksiyonları

Öncelikle işimize yarayacak bazı fonksiyonları görelim. Nix dilinde, listelerle çalışmak için bir dizi kullanışlı built-in fonksiyon bulunmaktadır. İşte Nix dilinde sıkça kullanılan bazı list fonksiyonları:

### `length` (Liste Uzunluğu)

Bir listenin eleman sayısını döndürür.

```nix
   let
     mylist = [1 2 3 4 5];
   in
     builtins.length mylist
  # result 5
```

### `map` (Liste Elemanlarına fonksiyon Uygulama)

Bir listenin her elemanına belirli bir işlevi uygular ve sonuçları içeren yeni bir listeyi döndürür.

```nix
   let
     mylist = [1 2 3 4 5];
   in
     map (x: x * 2) mylist
  # result [ 2 4 6 8 10 ]
```

### `filter` (Liste Elemanlarını Filtreleme)

Bir koşulu sağlayan liste elemanlarını içeren yeni bir liste döndürür.

```nix
   let
     mylist = [1 2 3 4 5];
   in
     filter (x: x % 2 == 0) mylist
```

### `++ ile` (Listeleri Birleştirme)

Birden fazla listeyi birleştirir.

```nix
   let
     list1 = [1 2 3];
     list2 = [4 5 6];
   in
     concatLists = list1 ++ list2
```

### `sort` (Listeyi Sıralama)

Bir listenin elemanlarını sıralar.

```nix
   let
     mylist = builtins.sort builtins.lessThan [ 483 249 526 147 42 77 ];
   in
     sort mylist
# [ 42 77 147 249 483 526 ]
```

### `builtins.tail` fonksiyonu

Listedeki ilk elema hraiş diğer elemenları alarak bir liste oluşturur.

```nix
builtins.tail [1 2 3 4 5]
# [ 2 3 4 5 ]
```

### `builtins.head` fonksiyonu

Listedeki ilk elemanı döndürür.

```nix
builtins.head [1 2 3 4 5]
# 1
```

### `builtins.concatLists` (listeleri birleştime)

2 veya daha fazla listeden tek bir liste oluşturur.

```nix

let
  list1 = [1 2 3];
  list2 = [4 5 6];
  list3 = [7 8 9];
  combinedList = builtins.concatLists [list1 list2 list3];
in
  combinedList

```

### `builtins.elemAt` (indekse göre listeden eleman getirmek)

```nix

let
  list = [1 2 3];
  result = builtins.elemAt list 2;
in
  result # 3

```

### `builtins.elem` (ile listede eleman aramak)

verilen değerri liste içinde arar ve bulursa true döndürür.

```nix

let
  list = [1 2 3];
  result = builtins.elem 2 list;
in
  result # true

```

### `builtins.genList` (ile liste oluşturmak)

verilen fonksiyonu kullanarak bir liste oluşturur.

```nix

let

list = builtins.genList (x: x * 2) 5;

in

list # [ 0 2 4 6 8 ]

```

### `builtins.groupBy` (ile liste elemanlarını gruplamak)

verilen fonksiyondan dönen string değerlerine göre liste elemanlarını gruplar. İlk parametre olan fonksiyon alttaki örnekte listedeki her bir elemanın 2. ve 3. karakterlerini alıyor ( ar, az ve oo). GroupBy fonksiyonu da içinde bu iki karaktere sahip olanları aynı gruba dahil ediyor.

```nix
mySet = builtins.groupBy (builtins.substring 1 2) ["foo" "bar" "baz"]

mySet
# { b = [ ... ]; f = [ ... ]; }

mySet.b
# [ "bar" "baz" ]

mySet.f
# [ "foo" ]


```

### `builtins.sort` (ile listelerde sıralama yapmak)

İlk parametre elemanları karşılaştıran fonksiyon ikinci parametre ise liste.

```nix
myList = builtins.sort builtins.lessThan [ 483 249 526 147 42 77 ]

:p myList
# [ 42 77 147 249 483 526 ]

```

### `builtins.map`

Bir liste elemanlarına verilen fonksiyonu uygular.

```nix
myList = map (x: "foo" + x) [ "bar" "bla" "abc" ]
:p myList
# [ "foobar" "foobla" "fooabc" ]
```

### `builtins.foldl’`

Bu fonksiyon, bir listenin veya başka bir veri yapısının her elemanı üzerinde bir işlem gerçekleştirerek sonuç elde etmeyi sağlar. foldl, soldan sağa doğru birleştirme işlemi yapar. Modern programlama dillerindeki reduce fonksiyonunun yaptığını yapar.

```nix


result = builtins.foldl' (x: y: x + y) 0 [1 2 3]
:p result
# 6

```

## Temel Set Fonksiyonları

En çok kullanılan set fonksiyonlarına bakalım.

### `builtins.hasAttr`

Belirli bir özelliğin (attribute) bir kümede bulunup bulunmadığını kontrol eder.

```nix
let
  mySet = { foo = "bar"; baz = 42; };
in
  builtins.hasAttr "foo" mySet  # true
```

### `builtins.attrNames`

Bir kümenin tüm özellik isimlerini içeren liste döndürür.

```nix
let
  mySet = { foo = "bar"; baz = 42; };
in
  builtins.attrNames mySet  # ["foo" "baz"]
```

### `builtins.attrValues`

Bir kümenin tüm özellik değerlerini içeren liste döndürür.

```nix
let
  mySet = { foo = "bar"; baz = 42; };
in
  builtins.attrValues mySet  # ["bar" 42]
```

### `builtins.elem`

Bir değerin bir küme içinde bulunup bulunmadığını kontrol eder.

```nix
let
  mySet = { foo = "bar"; baz = 42; };
in
  builtins.elem "bar" (builtins.attrValues mySet)  # true
```

### `builtins.filterAttrs`

Bir kümedeki özellikleri belirli bir koşula göre filtreler.

```nix
let
  mySet = { foo = "bar"; baz = 42; qux = true; };
in
  builtins.filterAttrs (name: value: value == true) mySet  # { qux = true; }
```

### Set'in eleman sayısını almak

Bunun için doğrudan bir fonksiyon yok ancak buraya kadar öğrendiklerimizle bunu yapabiliriz.
Öncelikle setin özelliklerini (attribute) alıp bunları bir liste haline getirip daha sonra bu listenin uzunluğunu alabiliriz.

```nix
let
  mySet = { foo = "bar"; baz = 42; };
  myList = builtins.attrNames mySet;
in
  builtins.length myList  # 2
```

### builtins.zipAttrsWith

Verilen fonksiyonu verilen set'lere uygularak yeni bir set oluşturur.

Alttaki örnekte set'lerin name'ine göre

```nix

mySet = builtins.zipAttrsWith (name: vals: {name=name; vals=vals;}) [{ a = 1; b = 1; c = 1; }  { a = 10; }  { b = 100; } ]

:p mySet

# { a = { name = "a"; vals = [ 1 10 ]; }; b = { name = "b"; vals = [ 1 100 ]; }; c = { name = "c"; vals = [ 1 ]; }; }

```

Aynı örneği alttaki gibi de yazabilirdik.

```bash
mySet = builtins.zipAttrsWith (name: values: { inherit name values; }) [{ a = 1; b = 1; c = 1; }  { a = 10; }  { b = 100; } ]

:p mySet

# sonuç
{ a = { name = "a"; values = [ [ 1 2 3 ] "test1" ]; }; b = { name = "b"; values = [ [ 1 2 3 ] "test2" [ "4" "5" "6" ] ]; }; c = { name = "c"; values = [ [ 7 "sekiz" 9 ] 9 ]; }; }

```

Aslında çok daha karmaşık zip fonksiyonları da yazılabilir. Ama burada o kadar derine inmeyeceğiz. Bu konuda yani zip fonksiyonları anlama konusunda diğer dillerden de örnekleri inceleyebilirsiniz. Ne yazık ki Nix dilinde yazılmış çok örnek yok. Karmaşık örnekleri bir sonraki makalemizde yapacağız. Bunun için daha fazla fonksiyonlar hakkında daha çok şey öğrenmeliyiz.

8. **builtins.removeAttrs (ile set'den eleman silme)**

Verilen listedeki elentleri set name'leri içinde arar ve bulduğu elementi listeden siler.

```nix
mySet = removeAttrs { x = 1; y = 2; z = 3; } [ "a" "x" "z" ]
:p mySet
# { y = 2; }
```

### builtins.mapAttrs

Bir fonksiyonu set'deki elamanlara uygular.

```nix
mySet = builtins.mapAttrs (name: value: value * 10) { a = 1; b = 2; }
p: mySet
# { a = 10; b = 20; }
```

## String ile ilgili fonksiyonlar

### `toString` (ile değerleri string'e çevirmek)

Bir değer'e string'e çevrilirken

- Path tipindekiler adres değişmeden string'e çevrilir (örneğin toString /foo/bar, "/foo/bar" şeklinde çevrilir.
- Eğer bir set `{ __toString = self: ...; }` veya `{ outPath = ...; }`şeklinde bir elamana sahipse ve bu elemanın içeriği string ise onu döndürür.
- Rakamlar olduğu gibi string'e çevrilir
- Liste elemanları doğrudan string'e çevrilerek yine lite döndürür.
- Bool değerlerşde eğer false ise boşluk değise string olarak 1 değeri döner.
- null ise boş string döndürür

```nix

mySet = { a = 1; b = 2; __toString = self:"merhaba";}
toString mySet
# result "merhaba"
```

### `builtins.substring` (ile bir string ifadeden parça almak)

İlk sayı başlangıç index'ini ikinci rakam ise bitiş index'ini gösterir.

```nix
result = builtins.substring 8 12 "Merhaba Dünya"
:p result
# Dünya

```

### `builtins.stringLength` (ile string'in uzunluğunu almak)

```nix
result = builtins.stringLength "Merhaba Dünya"

:p result
# 14
```

### `builtins.replaceStrings` (ile bir string ifadeyi değiştirmek)

Listeler halinde aranacak string ifade ve yerine geçecek string ifadedeler verilir. En sona da üzerinden işlem yapılacak string ifade verilir.

```nix
result = builtins.replaceStrings ["oo" "a"] ["a" "i"] "foobar"
:p result
# fabir

```

## `is` ile başlayan fonksiyonlar

Aslında bu fonksiyonları belki ilgili veri tipinin altına koymak mantıklı olabilir. Ancak akılda kalması için tek başlıkta listelemek mantıklı geldi.

- builtins.isString
- builtins.isPath
- builtins.isNull
- builtins.isList
- builtins.isInt
- builtins.isFunction
- builtins.isFloat
- builtins.isBool
- builtins.isAttrs

Bu fonksiyonların çalışma mantığı hep aynı, fonksiyona verilen değerin kontrol edilen tipten olup olmadığını kontrol ederler.

```nix
result = builtins.isList "merhaba"

:p result

# result false

```

## Fetch ile Başlayan Fonksiyonlar

- **builtins.fetchGit**: Git repolarından dosya indirmek için kullanılır.
- **builtins.fetchClosure**: Nix store'da bulunan bir dosyayı indirmek için kullanılır.
- **builtins.fetchTarball**: Tarball dosyalarını indirmek için kullanılır.
- **builtins.fetchurl**: URL'den dosya indirmek için kullanılır.

Diğer fetcher'lar için [şu linke](https://ryantm.github.io/nixpkgs/builders/fetchers/) ve [resmi NixOs sayfasına](https://nixos.org/manual/nixpkgs/stable/#chap-pkgs-fetchers) göz atabilirsiniz. NixOs sayfasındakiler sadece builtins fonksiyonlarını içermiyor.

Biz aslında nixpkgs reposunu kullandığımızda repo bir tar dosaysı oalrak sistemimize indirilir. Extract edildikten sonra lokal dosyalarla işlem yapılır.

## Loop Kullanımı

Daha önce de bahsettiğimiz gibi Nix dilinde loop yoktur. Ancak recursive fonksiyonlar kullanarak döngü mantığını oluşturabiliriz. Bir önceki makalede henüz modülleri görmediğimiz için bu konuya değinmemiştik. Artık builtins fonksiyonlara değindiğimize göre bu konuya tekrar dönebiliriz.

Loop kullanımında map, fold ve filter gibi fonksiyonlar çok işimize yarayacak. Sonuçta amacımız liste elemanlarında dolaşmak ve elamanlar üzerinde işlemler yapabilmek. Bu fonksiyonlarda bunu bize sağlayacak. Bir iki örnek yapalım.

Bir nix dosyası oluşturup içine alttaki kodları kopyalayalım. Amacımız bütün elemanları 2 ile çarpmak.

```nix
let
myList = [1 2 3 4 5];
result = builtins.map (x : x * 2) myList;
in
{
inherit result;
}
```

repl'i açıp alttaki komutu çalıştıralım.

```bash
:l ./loop-example1.nix

:p result.result
# [ 2 4 6 8 10 ]

```

Bir örnek daha yapalım. Bu sefer iki listenin elemanlarını birbiri ile çarpıp yeni bir liste oluşturalım. Eğer salt fonksiyonel olmayan bir dilde kod yazıyor olsaydık bu işlem için iki tane döngü kullanmamız gerekirdi. Ancak Nix dili daha önce bahsettiğimiz gibi fonksiyonel bir dil olduğu için biraz farklı düşünmemiz gerekiyor. bu arada tabii ki bu yaptığımızı builtins veya ileride göreceğimiz lib modülü içindeki fonksiyonlarla yapabilirsiniz. Yani iki listedeki elemanları birbiriyle çarpmanın başka yolları da var ancak bizim amacımız hakikaten 2 liste elamanlarını iç içe gezmemiz gerekir ne yapabiliriz onu görmek. Birazda tabi beyin jimnastiği yapmak.

```nix
let

list1 = [1 2 3 4];
list2 = [5 6 7 8];


mul = my-list1: my-list2:
                if builtins.length my-list1 > 0 && builtins.length my-list2 > 0
                    then [(builtins.head my-list1 * builtins.head my-list2)] ++ mul (builtins.tail my-list1) (builtins.tail my-list2)
                else
                    [];
myList = mul list1 list2;

in
{
myList = myList;
list1 =  list1;
list2 =  list2;
}
```

Yine recursive fonksiyondan faydalanıyoruz. Head fonksiyonu listenin ilk elemanını alırken tail fonksiyonu listenin ikinci elemanında başlayarak son elemana kadar bütün elemanları bir liste olarak döndürür. Yaptığımız recursive fonksiyon aslında şöyle bir şey yapıyor.

```nix

list1 = [1 2 3 4];
list2 = [5 6 7 8];

sum = [1 * 5] ++ [2 * 6] ++ [3 * 7] ++ [4 * 8] ++ [];

```

Yazdığıız kodu bir dosyaya kaydedip repl'de çalıştıralım.

```bash
 :l ./loop-example2.nix
# Added 3 variables.
myList
# [ 5 12 21 32 ]
```

Şimdilik bu kadar. Ancak Nix'de tabii ki bundan daha fazlası var ancak baya bi işimizi görecek kadarlık kısmını görmüş olduk. Bundan çok daha fazlası da nixpkgs içindeki lib modülünde var.

Umarım faydalı olmuştur.

Bir sonraki makalemizde Nix paket yöneticisine geçeceğiz. Zaten bundan sonra buraya kadar gördüklerimizin bol bol uygulamasını yapıyor olacağız.

## Referanslar

- [https://nix.dev/tutorials/nix-language.html#lookup-paths](https://nix.dev/tutorials/nix-language.html#lookup-paths)
- [https://nix.dev/tutorials/module-system/module-system](https://nix.dev/tutorials/module-system/module-system)
- [https://nixos.org/manual/nixpkgs/stable/](https://nixos.org/manual/nixpkgs/stable/)
- [https://nixos.org/manual/nix/stable/command-ref/env-common.html](https://nixos.org/manual/nix/stable/command-ref/env-common.html)
- [https://nixos.wiki/wiki/Nix_by_example#Exceptions](https://nixos.wiki/wiki/Nix_by_example#Exceptions)
- [https://nixos.wiki/wiki/Nix_by_example#Assertions](https://nixos.wiki/wiki/Nix_by_example#Assertions)
- [https://nixos.wiki/wiki/Nix_by_example#Debugging](https://nixos.wiki/wiki/Nix_by_example#Debugging)
- [https://teu5us.github.io/nix-lib.html](https://teu5us.github.io/nix-lib.html)
- [https://fasterthanli.me/series/building-a-rust-service-with-nix/part-10](https://fasterthanli.me/series/building-a-rust-service-with-nix/part-10)
- [https://ertt.ca/nix/shell-scripts/](https://ertt.ca/nix/shell-scripts/)
