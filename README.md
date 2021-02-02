<p align="center">
  <img src="https://engeto.cz/wp-content/uploads/2019/01/engeto-square.png" width="200" height="200">
</p>

# Java 2 - 4. lekce

V dnešní lekci si vysvětlíme vyjímky a jejich použítí v Javě. A projdeme si zadání dvou projektů pro kurz Java 2.

## Vyjímky

Tato kapitola navazuje na předchozí části kurzu, kde jsme se seznámili s řízením toku programu pomocí `if-else`, `do-while`, `switch`, `break` a podobné. Dosud jsme v kurzu pracovali jen se situacemi, kdy program buď fungoval správně nebo nefungoval vůbec. Například, když jsme zkoušeli dělit nulou, tak program skončil chybou `ArithmeticException` a vypsal:

```
public class DeleniNulou {
    public static void main(String[] args) {
        int c = 1 / 0;
    }
  }
```

```
Exception in thread "main" java.lang.ArithmeticException: / by zero
    at DeleniNulou.main(DeleniNulou.java:3)
```

Výraz `Exception / Vyjímka` je z výrazu Exceptional event / Výjímečná událost.

Definice: Vyjímka je událost, která nastane pří vykonávání programu a naruší běžný chod programových instrukcí.

Při vývoji software je potřeba počítat i se situacemi, kdy některá volání mohou dopadnou pokaždě různě a v případě chyby je potřeba patřičně zareagovat. Například, když uživatel zkouší uložit data na disk, který právě odpoji nebo pokud zkouší stáhnout data ze serveru, ale spojení je přes nestabilní Wi-Fi. Řídít takovou logiku pomocí větvení je sice možné, ale není to příliš praktické.

Ukázka kódu s výjimkou
```
try {
    //Nejaka operace, kde muze nastav vyjimka
} catch (Exception e){
    e.printStackTrace();
}
```

### Stack volání metod

Ukázka StackTrace při chybě

![Image](Drawing1.png)

```
Exception in thread "main" java.lang.NullPointerException
        at com.example.myproject.Book.getTitle(Book.java:16)
        at com.example.myproject.Author.getBookTitles(Author.java:25)
        at com.example.myproject.Start.main(Start.java:14)
```

```
class ThrowException{
  public void run() throws Exception{
    System.out.println("ThrowException");
    throw new Exception("Chyba");
  }
}
class WithoutExceptionHandler{
  public void run() throws Exception{
    System.out.println("WithoutExceptionHandler");
    ThrowException t = new ThrowException();
    t.run();
  }
}
class WithExceptionHandler{
  public void run() {
    System.out.println("WithExceptionHandler");
    WithoutExceptionHandler wo = new WithoutExceptionHandler();
    try{
      wo.run();
    }catch(Exception e){
      System.out.println("Exception:"+e);
    }
  }
}
class Main {
  public static void main(String[] args) {
    System.out.println("main");
    WithExceptionHandler wi = new WithExceptionHandler();
    wi.run();
  }
}
```

```
java -classpath .:/run_dir/junit-4.12.jar:target/dependency/* Main
main
WithExceptionHandler
WithoutExceptionHandler
ThrowException
Exception:java.lang.Exception: Chyba
```

### Použití výjimek

#### Diagram hierarchie vyjimek v JDK
![Image](Drawing2.png)

### Základní typy Excepetions v JDK

Rozdíl mezi Checked, Unchecked výjimkami a Error

V Javě jsou definováné dva různé typy vyjímek a jejich chování. Checked (hlídané), u kterých je potřeba definovat chování explicitně v programu a kompilátor to kontroluje. Unchecked (nehlídané nebo běhové) vyjímky jsou potomky tříd java.lang.Error nebo java.lang.RuntimeException a takové vyjímky není nutné definovat v programu. Je možné je odchytit, ale často jde o takové chyby, kde už není možné a ani žádoucí v běhu programu pokračovat, např java.lang.OutOfMemoryError. Pokud taková chyba nastane tak je často potřebat změnit logiku programu (použít jiný algoritmus), změnit nastavení (alokovat více paměti pro JVM) nebo program přesunout na systém s více paměti


### Použití výjimek

Vyjímka má několik částí

- try - klíčové slovo na začátku bloku
- try (definice AutoCloseable zdroje) - volitelný blok pro definici zdrojů, které budou automaticky uzavřené a není nutné je uzavírat ve finally bloku
- try {} - hlavní blok kódu, který miže vyvolat vyjímku
- catch (definice vyjímky) - definice typu vyjímky k ošetření
- catch (..) {ošetření vyjímky} - blok kódu na ošetření vyjímky
- finally {} - blok kódu, který se provede v každém případě


#### Try-Catch
```
try {
   statements-1
}
catch ( exception-class-name  variable-name ) {
   statements-2
}
```
#### Multi-catch blok
```
try {
   statements-1
}
catch ( exception-class-name  variable-name ) {
   statements-2
} catch (other-exception-class-name variable-name) {
    statements-3
}
```
Je možné seskupit definici vyjímek dohromady

```
try {
   statements-1
}
catch ( exception-class-name | other-exception-class-name  variable-name ) {
   statements-2
} catch (another-exception-class-name variable-name) {
    statements-3
}
```
#### Try-Catch-Finally
```
try {
   statements-1
}
catch ( exception-class-name  variable-name ) {
   statements-2
} finally {
    statements-3
}
```
#### Try-with-resources
Specialní konstukt v Javě značně zjednodušuje práci se I/O je try-with-resources. Tento konstrukt umožní vytvořit instance objektů java.lang.AutoCloseable bez potřeby definovat jejich uzavření. Práce se soubory je tímto mnohem jednodušší a je možné vyvarovat se chyb, kde soubory jsou programem uzamčené až do ukončení běhu programu. Více detailů si ukážeme v další kapitole vysvětljující práci s I/O.

```
try (create-autocloseable-resources-statements) {
    statements-1
} catch(SomeOtherException ex) {
    statements-2
}
```

### Časté příklady výjimek

#### NullPointerException (NPE)

Jedna z velice častých chyb je přístup na metody objektu, který není inicializovaný. Např chybějící konfigurace, nezadaná hodnota od uživatele nebo špatně strukturovaný kód, porovnání pomocí equals(...) nebo volání toString()

```
String valueA;
String valueB;
// CHYBA - Zde nastavane NullPointer Exception
if (valueA.equals(valueB)){
    //
}
```

Do verze JDK 14

```
Exception in thread "main" java.lang.NullPointerException
    at MyClass.main(MyClass.java:8)
```
Od verze JDK 14
```
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "java.lang.Long.equals(Object)" because "<local1>" is null
    at Main.main(Main.java:8)
```

### Řízení toku programu pomocí výjimek

#### Vyvolání výjimky pomocí Throw
Vyvolání výjimky se dělá pomocí klíčového slova throw a následuje objekt odvozený od Exception. Většinou se ten objekt rovnou vytváří throw new Exception("some message");

```
throw someThrowableObject;
```
```
public void someMethod(int size) {
    if (size < 0) {
        throw new InvalidArgumentExecption("Size have to be non-negative");
    }
}
```
#### Klíčové slovo Throws

Pro Checked (hlídané) výjimky je potřeba definovat u metod, který typ výjimky může nastat. Toto se dělá pomocí klíčového slova throws uvedeného mezi definicí metody a jejím tělem.

```
public void thisMethodMayThrowException() throws TypeOfTheExceptionThrown {
   ... tělo metody, kde může vyjímka nastat ...
}
```

Ukázka volání vnořené výjimky:

1. Vyvolání výjimky
2. Odchycení v Catch bloku
3. Vyvolání runtime výjimky (neošetřená)
4. Finally blok
5. Předání výjimky dál - v tomto případě pád programu a výpis StackTrace

```
class First {
  void run() throws Exception{
    System.out.println("First run");
    throw new Exception("Error");
  }
}
class Second {
  void run(){
    First f = new First();
    try{
      f.run();
    }catch(Exception e){
      System.out.println("Second catch");
      throw new RuntimeException("Crash");
    }finally{
      System.out.println("Second finally");
    }
  }
}
class Main {
  public static void main(String[] args) {
    Second s = new Second();
    s.run();
  }
}
```

Výsledek:
```
First run
Second catch
Second finally
Exception in thread "main" java.lang.RuntimeException: Crash
    at Second.run(Main.java:15)
    at Main.main(Main.java:25)
```

### Dědičnost a Throws

Je možné definovat vlastní definice výjimek, které se navzájem řetězí a dědí od sebe. Takové řetězení má svoje pravidla a překladač kontrolu, že nejsou definované nedosažitelné catch bloky.

- Výjimka je zachycena nejbližším vhodným catch blokem
- Speciálnější výjimka musí být uvedena před obecnější
- V kódu nejsou nedosažitelné bloky, kde jsou speciálnější výjimky uvedeny později

#### Není možné definovat obecnější výjimku před specifičtější
```
class MyException1 extends Exception {}
class MyException2 extends MyException1 {}
class Main {
  public static void main(String[] args) {
    int x = 1;
    try {
      if (x == 1) {
        throw new MyException1();
      } else if (x == 2) {
        throw new MyException2();
      }
      // Obecnější výjimka MyException1 má přednost
    } catch (MyException1 mv) {
   //CHYBA:
   // Speciálnější výjimka MyException2 nikdy nebude zachycena, proto tuto část překladač nepovolí
    } catch (MyException2 mv) {
    }
  }
}

```

```
 javac -classpath .:/run_dir/junit-4.12.jar:target/dependency/* -d . Main.java
Main.java:18: error: exception MyException2 has already been caught
    } catch (MyException2 mv) {
      ^
1 error
compiler exit status 1
```

### Hlavní výhody použití výjimek
- Oddělení programové logiky pro ošetření chyb od hlavní logiky
- Řetězení vyjímek a propagování vyjímky v Call Stack na místo, kde je řešení nejefektivnějsí
- Možnost definovat a seskupit různé typy chyb dohromady
- Užitečný reporting chyb

#### Praktické použití

Dosud jsme použili e.printStackTrace() k vypsání chybové hlášky do konzole. Jde o jednoduché a efektivní řešení pokud aplikaci ladíme na vlastním počítači. V případě serverové aplikace tento přístup není vhodný hned z několika důvodů.

1. Logování so System.out a System.err není úplně flexibilní
2. Chybí identifikace, kdy k chybě došlo
3. Když už jsme vyjímku zachytili, tak je občaš vhodné doplnit další data
4. Chybí identifikace závažnosti chyby
Pro tento případ existuje hned několik Frameworků, které zjednodušují logování. Například Log4j, Log4j2, java.util.logging nebo SLF4J. Liší se v mnoha detailech a možnosti konfigurace. Ale při psaní kodu vypadají všechny relativně podobně.

```
import java.util.logging.*;
class Main {
  private static Logger logger = Logger.getLogger("com.test.logovani");
  public static void main(String[] args) {
        logger.fine("doing stuff");
        try {
            throw new Exception();
        } catch (Exception ex) {
            // Log the exception
            logger.log(Level.WARNING, "trouble ", ex);
        }
        logger.fine("done");
  }
}
```

```
> javac -classpath .:/run_dir/junit-4.12.jar:target/dependency/* -d . Main.java
> java -classpath .:/run_dir/junit-4.12.jar:target/dependency/* Main
Oct 17, 2020 9:30:55 PM Main main
WARNING: trouble
java.lang.Exception
    at Main.main(Main.java:8)
```

### Příklady k procvičení

#### 1 Vyvolání výjimky
```
Napiště jednoduchý program, který vyvolá výjimku.
Např: Přístup do pole mimo definovaný rozsah, dělení nulou, cast na špatný typ,..
```
#### 2 Vlastní výjimka
```
Napište program, který definuje vlastní výjimku (hlídané) a zkusí ji vyvolat
```
#### 3 Vyvolání nehlídané výjimky
```
Napiště program, který zkusí vyvolat nehlídanou vyjímku.
Např: Nekonečná rekurze, pokus alokovat více než povolené paměti, ...
```
#### 4 Řetězení výjimek
```
Napiště program, který definuje 2 typy výjimek. Definuje metodu, která v sobě bude řetězit výjimky.
1. Vyvolá výjimku prvního typu
2. Zachytí ji
3. Vyvolá výjimku druhého typu
4. Nechá program skončit vypsáním StackTrace
```


## HTTP Client v JDK

Od verze JDK 11 je v Javě nové API pro volání HTTP volání. Můžete najít návody na použití `HttpUrlConnection`, které je v Javě od prvních verzí. Pro vás je lepší se tomu vyhnout.

### Co to je HTTP Klient a k čemu to slouží
HTTP je protokol pro Client-Server komunikaci přes internet. Skoro všechna internetová komunikace to využívá.

- URL
- Metoda
- Hlavička
- Tělo

Metody
- GET - Jenom URL
- POST - URL a tělo zprávy
- PUT - URL a tělo zprávy (idempotentní - více volání skončí stejným stavem)
- DELETE - URL a tělo zprávy
- PATCH - Slouží k částečné upravě data
- OPTION - Pomocné volání

### REST API
- Služba která s kterou je možné komunikovat pomocí HTTP
- API Volání dodržují několik pravidel

### Použití

#### Inicializace klienta
```
HttpClient httpClient = HttpClient.newBuilder()
              .version(Version.HTTP_2)  // this is the default
              .build();
```

#### Vytvoření GET requestu
```
HttpRequest request = HttpRequest.newBuilder()
               .uri(URI.create("https://https.github.io/"))
               .GET()   // this is the default
               .build();
```

#### Vytvoření POST requestu
```
HttpRequest mainRequest = HttpRequest.newBuilder()
               .uri(URI.create("https://http2.github.io/"))
               .POST(BodyPublishers.ofString(json))
               .build();
```

#### Poslání requestu a zpracování výsledku
```
HttpResponse response = httpClient.send(request, BodyHandlers.ofString());
logger.info("Response status code: " + response.statusCode());
logger.info("Response headers: " + response.headers());
logger.info("Response body: " + response.body());
```

#### Poslání requestu a asynchroní zpracování výsledku
```
httpClient.sendAsync(request, BodyHandlers.ofString())
          .thenAccept(response -> {
       logger.info("Response status code: " + response.statusCode());
       logger.info("Response headers: " + response.headers());
       logger.info("Response body: " + response.body());
});
```
