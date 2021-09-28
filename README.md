# Tersine Mühendisliği Anlamak

Derlenmiş ve kapalı kaynak bir kodu birtakım tersine mühendislik araçları kullanarak assembly kodunu inceleyip temel felsefeyi anlamaya çalışacağız.

İlk olarak bir C kodunun assembly kodlarını elde etmemiz gerekmektedir. Assembly kodlarını elde etmek, incelemek ve anlayabilmek için https://godbolt.org/ web sitesinden yararlanacağız. 

## Makine Dilinde Koşullar

Koşul ifadesi (if, else if, for gibi) barındıran bir durum makine dilinde CMP komutu ile sağlanır. Yani makine kodunda CMP ifadesi ve alt satırında jle, jns gibi 
bayraklar içeriyorsa (bknz : https://www.tutorialspoint.com/assembly_programming/assembly_conditions.htm) burada bir koşul olduğundan bahsedebiliriz. 

### Makine Dilinde *if* ve *else if* Koşullarını Anlamak

C kodları bildiğiniz üzere derlenmeye ihtiyaç duyarlar. Bu seride herhangi bir C kodunu derlemek için linux üzerindeki gcc derleyicisini kullanacağım. C kodunu yazdıktan sonra .c uzantılı olarak kaydedip Linux terminaline ``` gcc dosya_adı.c -o dosya_adı``` yazarak derliyoruz.

Derlediğimiz C kodunu GHIDRA aracı ile analiz edelim. GHIDRA aracını nasıl kullanacağınızı bilmiyorsanız "https://www.youtube.com/watch?v=fTGTnrgjuGA" bu videoya göz atabilirsiniz.

Tersine mühendislik yaptığımız herhangi bir yazılımda if, else if, switch case gibi koşullu ifadeleri bulunabilir. Ve bu ifadeleri birbirinden ayırabilirsek kodun arka planını daha net görebiliriz. Peki makine diline bakarak if ve else if durumlarını birbirinden ayırt edebilir miyiz?

Üst üste yazılmış 3 if koşul yapısında bir göz atalım. 

```C
#include <stdio.h>

int main()
{
    int number = 5;
    if(number < 0) 
    {
        printf("This number is negative");
    }
    if(number > 0) 
    {
        printf("This number is positive");
    }
    if(number == 0) 
    {
        printf("This number is 0");
    }

    return 0;
}
```

![](https://user-images.githubusercontent.com/58151582/134725523-6ded2bca-d1c0-4c6b-8d27-c93d67b742bd.png)

Ekran alıntısında da görüldüğü üzere if komutu makine diline CMP ve jump instructions ile çevirilir. Elde ettiğimiz sonuç çok şaşırtıcı değil. 

Şimdi else if komutunun makine koduna bakalım. 

```C
#include <stdio.h>

int main()
{
    int number = 5;
    if(number < 0) 
    {
        printf("This number is negative");
    }
    else if (number > 0) 
    {
        printf("This number is positive");
    }
    else if (number == 0) 
    {
        printf("This number is 0");
    }

    return 0;
}
```

![](https://user-images.githubusercontent.com/58151582/134725693-34bc87e8-14df-41a2-b705-4058bc1583b9.png)

Sanırım işler biraz karıştı. Sağduyumuza göre else if komutunun if komutundan farklı bir yapıya çevirilmesi bekleriz. Ama malesef makine diline çevirilirken
if ile else if aynı offsete dönüştürülür. 

Bir de yazdığımız bir C kodunu GHIDRA aracı ile reverse ederek elde ettiğimiz C kodunu inceleyelim. 

```C
#include <stdio.h>

void main()
{
	int number;
	
	printf("pls type a number : \n");
	scanf("%d",&number);
	if (number < 0)
	printf("Number is negative");
	
	else if (number > 0)
	printf("Number is positive");
	
	else
	printf("Merhaba ben volkan konak");
}
```

| KAYNAK KOD | GHIDRA Decompile|
| --- | --- |
|![](https://user-images.githubusercontent.com/58151582/134726918-8ea10a07-c175-4803-91b8-2142bf54043d.png) | ![](https://user-images.githubusercontent.com/58151582/134727203-f5938686-7734-4d11-81b4-92ad8a58fa96.png) |

Assembly kodundan kaynak koda giderken else if koşul komutu decompile tarafında else if olarak değil else ifadesinin altında bir if else yapısı olarak çeviriyor. GHIDRA analizimiz de bize gösteriyor ki if ve else if komutlarını birbirinden ayırmak pek mümmkün değil. 

Sağ taraftaki kodun 0 ile 1 arasındaki durumlarda yanlış çıktı vereceğini düşünebilirsiniz ama değeri integer olarak alığımızı unutmayın :)

### Makine Dilinde *for*, *while* ve *if* Koşullarını Anlamak 

if ve else if de olduğu gibi while ve for döngüleri de birer koşul durumu ile çalışmaktadırlar. Peki bir defa çalışan for ve while döngüsünü bir if koşulundan nasıl ayırabiliriz. 

```C
#include <stdio.h>

int main()
{
    int i = 1;
    if(i < 2) 
    {
        printf("zaa");
    }
    for (; i < 2; i++) 
    {
        printf("%d",i);
    }
    while(i < 10) 
    {
        printf("sa");
        i++;
    }

    return 0;
}
```

![](https://user-images.githubusercontent.com/58151582/134745654-deb3138d-ee1d-4b1c-b0e0-64404d64c379.png)

if komutu içerisindeki şart geçerli olduğu zaman cmp nin altındaki flagin (jg,je,jle gibi) karşısındaki komut setine gidilir. 
Karşılaştırma şartı uygun değilse kod akışı devam eder ve if içerisindeki ilgili offset if karılaştırmasının (cmp ve ilgili jump instructions) altındadır.

![](https://user-images.githubusercontent.com/58151582/134746043-ad4b8f4d-274c-4e15-bd38-1d4995be0f91.png)

Ama for döngüsünün iç kısmı assembly karşılaştırılmasının üst kısmında kümelenir ve en alt satırında add komutu yer alır. While döngüsü ile for döngüsü de aynı yapıdadır ama add komutu içermez (kasti olarak while içerisinde kontrol değişkeni değiştirilmiyorsa)

Yani tek seferlik çalışan bir while ve for döngüsü ile if arasındaki fark kolayca anlaşılabilir. Yukarıdaki ekran alıntısından da anlaşıldığı gibi for ve while döngüsü de if, else if yapısında olduğu gibi aynı assembly komut setine sahiptir. Peki while ile for arasındaki farkı nasıl anlayabiliriz?

Bunun için https://crackmes.one/ sitesi üzerindeki bir crackmes'i analiz edeceğiz. GHIDRA ve IDA PRO araçlarını kullanarak decompile yapalım. (IDA PRO decompile çıktısı ilgili crackmes'in çözümlerinden alınmıştır.)

GHIDRA ile analiz ettiğim crackmes'i https://crackmes.one/crackme/612e85d833c5d41acedffa4f bu bağlatıdan indirebilir ve sonuçları kendiniz de test edebilirsiniz.
zip dosyasını parolası : crackmes.one

Analiz ederken bazı değişkenleri ve kullanıldıkları yerleri daha rahat görmek açısından değişkenlere yeni isimler verdim. Bundan dolayı analizi yaptığınızda değişken adları bu ekran alıntısındaki ile aynı olmayacaktır. Örneğin, scanf ile alınan bir veri sizde "local_48" olarak adlandırılmış olabilir. Onu "user_name" olarak değiştirdim. Gerekli değişken isimlendirmelerini kod yapısını inceleyerek ve mantıksal çıkarımlarla isimlendirebilirsiniz. İlgili değişkene sağ tıklayarak "rename variable" sekmesine tıklayın ve değiştirin.  

Değişkenleri el ile tek tek değiştirmemek için sol üstteki edit sekmesine tıklayın. "Tool options" sekmesine tıklayın ve karşınıza çıkan ekranın alt kısmındaki filtreye "cursor text highlight" yazın. "Mouse button to active" yazan satır sizde "Middle" olarak seçili olacaktır. "Left" ile değiştirin ve uygulaya basın. Artık bir değişkenin ismini kodun her yerinde aynı anda değiştirebilirsiniz. 

| GHIDRA | IDA PRO |
| --- | --- |
| ![](https://user-images.githubusercontent.com/58151582/134747668-abdc0299-8836-4aa3-ae9d-8191a13af7cf.png) | ![](https://user-images.githubusercontent.com/58151582/134747711-930a8d60-99f6-4fef-9491-521ce260a2d4.png) |

GHIDRA ve IDA PRO ile aynı kodu analiz ederken bile decompilerlar ilgili döngü için farklı çıktılar vermektedir. IDA PRO ve GHIDRA'nın bizlere gösterdiği gibi kontrol değişkeni while içerisinde değiştirilmediği sürece while ve for döngüsünü birbirinden ayırmak çok zordur. 

Buradan anlaşılacağı üzere tersine mühendislik yaparken elde ettiğimiz kaynak kodu aslında ana kaynak kodun bir yansımasıdır. Her ne kadar decompilerlar tarafından bizlere verilen kaynak C kodları ana kod ile aynı işlevi görse de birebir olarak çeviri mümkün değildir.

## İşleri Biraz Karıştıralım

Tersine mühendislik yaptığımızda arka tarafta nelerin döndüğünü temel olarak anladığımızı düşünüyorum. Bir adım ilerisine geçerek makine dilinde fonksiyon kavramına bir bakış atalım. Bu konunun önemli olduğu kanısındayım. Çünkü tersine mühendislik yapacağınız bir çok uygulama fonksiyon yapısını barındırmaktadır. Bu olguyu anlayabilirsek bakış açımızı daha da genişletebiliriz.  

### Makine Dilinde Fonksiyonlar

```C
#include <stdio.h>
int func()
{
    printf("selam");
}
int main()
{
    func();

    return 0;
}
```
![image](https://user-images.githubusercontent.com/58151582/135108192-401fc322-5faf-4f2e-9243-40ead8317c1d.png)
 
Bir fonksiyon tanımlandığı zaman yukarıdaki ekran alıntısında da görüldüğü gibi her zaman fonksiyonun adıyla olmasada fonksiyon olduğunu belirten bir ad ile tanımlanır.

### İterative vs Recursive

Bu yazıda iterative ve recursive fonksiyon hakkında ufak bir hatırlatma yapalım. 

İterative (yinelemeli) fonksiyon : For, while ve do...while C'deki döngüler, bir koşul doğru olduğu sürece yürütülecek döngülerdir. Bu döngülerin bulunduğu foksiyonlara yinelemeli fonksiyon denir.

Recursive (özyinelemeli) fonksiyon : Kendi kendisini çağıran fonksiyonlara özyinelemeli fonksiyonlar denir. 

Şimdi de recursive ve iterative fonksiyonları makine dilinde karşılaştıralım.

Recursive 

```C
#include <stdio.h>

int factorial(unsigned int i) {

   if(i <= 1) {
      return 1;
   }
   return i * factorial(i - 1);
}

int  main() {
   int i ;
   printf("type a number : ");
   scanf("%d",&i);
   printf("factorial is %d\n", factorial(i));
   return 0;
}
```
İterative 

```C
#include <stdio.h>
int factorial(int n)
{
    int fact = 1 ;
    for(int i = 1; i <= n; ++i)
    fact = fact * i;

    return fact;
}
int main() {
int n = 5;

printf("Factorial for 5 is %d", factorial(n));
return 0;
}
```
| Recursive | İterative|
|---|---|
|![](https://user-images.githubusercontent.com/58151582/135175071-2fb3b8a4-fb2a-4a2e-9134-841331bdc6bb.png)|![](https://user-images.githubusercontent.com/58151582/135175152-0ab086bf-c168-4a03-9e76-878958d7c833.png)|

Recursive ve iterative fonksiyonların birbirlerine benzer ama kolayca ayırt edilebilir bir makine dili söz dizimine sahip olduğunu söyleyebiliriz. Örneğin; her ikisi de içerisinde cmp ve jump instructions yani bir karşılaştırma ifadesi barındırır, her ikisi de birer döngü durumudur. Ama aradaki farkı yukarıda bahsettiğimiz for durumu sayesinde kolayca görebiliriz. 

Peki yukarıdaki anlatıma ek olarak ghidra'nın bize verdiği kullanışlı bir özellik olan "function graph" özelliğini iki fonksiyon için kullanalım. Bu özellik sayesinde makine tarafında bu iki fonksiyonun farkını daha net görebiliriz. Kendiniz de denemek istiyorsanız Üst penceredeki window sekmesine tıklayarak bu özelliği ve daha fazlasını görebilirsiniz. 

| İterative | Recursive |
|---|---|
|![](https://user-images.githubusercontent.com/58151582/135178880-a858c3ed-2713-427e-9987-898fb3f3acfd.png)|![](https://user-images.githubusercontent.com/58151582/135178832-7f25acfb-377e-49b0-91d9-69d2d4960db9.png)|

Tersine mühendislik yaparak analiz ettiğimiz recursive ve iterative fonksiyonların birbirlerinden ne kadar da farklı yapılar olduklarını gösteren çok net bir görsel. Görüldüğü üzere recursive uygun şart sağlanana kadar sürekli kendisini çağırmaya devam ediyor. İterative bir fonksiyon ise içerisinde bulunan for döngüsünü gereği kadar döndürerek döngüden çıkıyor.  

