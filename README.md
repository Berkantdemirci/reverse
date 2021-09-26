# Tersine Mühendisliği Anlamak

Derlenmiş ve kapalı kaynak bir kodu birtakım tersine mühendislik araçları kullanarak assembly kodunu inceleyip temel felsefeyi anlamaya çalışacağız.

İlk olarak bir C kodunun assembly kodlarını elde etmemiz gerekmektedir. Assembly kodlarını elde etmek, incelemek ve anlayabilmek için https://godbolt.org/ web sitesinden yararlanacağız. 

## Makine Dilinde Koşullar

Koşul ifadesi (if, else if, for gibi) barındıran bir durum makine dilinde CMP komutu ile sağlanır. Yani makine kodunda CMP ifadesi ve alt satırında jle, jns gibi 
bayraklar içeriyorsa (bknz : https://www.tutorialspoint.com/assembly_programming/assembly_conditions.htm) burada bir koşul olduğundan bahsedebiliriz. 

### Makine Dilinde *if* ve *else if* Koşullarını Anlamak

Bir uygulamada if ve else if yapıları makinenin anlayabilmesi için assembly komut setine çevirilmelidir. Tersine mühendislik yaptığımız herhangi bir yazılımda da if, else if, switch case gibi koşullu ifadeleri bulunabilir. Ve bu ifadeleri birbirinden ayırabilirsek kodun arka planını daha net görebiliriz. Peki makine diline bakarak if ve else if durumlarını birbirinden ayırt edebilir miyiz?

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

C kodları bildiğiniz üzere derlenmeye ihtiyaç duyarlar. Bu seride herhangi bir C kodunu derlemek için linux üzerindeki gcc derleyicisini kullanacağım. 
C kodunu yazdıktan sonra .c uzantılı olarak kaydediyoruz. Linux terminaline ``` gcc dosya_adı.c -o dosya_adı``` yazarak derleyelim.
Derlediğimiz C kodunu GHIDRA aracı ile analiz edelim. GHIDRA aracının nasıl kullanacağınızı bilmiyorsanız https://www.youtube.com/watch?v=fTGTnrgjuGA bu videoya göz atabilirsiniz.

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

Bunun için crackmes.one sitesi üzerindeki bir crackmes'i analiz edeceğiz. GHIDRA ve IDA PRO araçlarını kullanarak decompile yapalım. (IDA PRO decompile çıktısı ilgili crackmes'in çözümlerinden alınmıştır.)

| GHIDRA | IDA PRO |
| --- | --- |
| ![](https://user-images.githubusercontent.com/58151582/134747668-abdc0299-8836-4aa3-ae9d-8191a13af7cf.png) | ![](https://user-images.githubusercontent.com/58151582/134747711-930a8d60-99f6-4fef-9491-521ce260a2d4.png) |

GHIDRA ve IDA PRO ile aynı kodu analiz ederken bile decompilerlar ilgili döngü için farklı çıktılar vermektedir. IDA PRO ve GHIDRA'nın bizlere gösterdiği gibi kontrol değişkeni while içerisinde değiştirilmediği sürece while ve for döngüsünü birbirinden ayırmak çok zordur.  

Buradan anlaşılacağı üzere tersine mühendislik yaparken elde ettiğimiz kaynak kodu aslında ana kaynak kodun bir yansımasıdır. Her ne kadar decompilerlar tarafından bizlere verilen kaynak C kodları ana kod ile aynı işlevi görse de birebir olarak çeviri mümkün değildir.

