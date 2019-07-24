---
layout: post
title:  "Spring Boot: Out of the Box vergleich"
date:   2019-07-22 13:48:04 +0200
categories: development kotlin java spring
---

**Dies ist der Start einer Blogserie, die sich mit den Unterschieden zwischen Java und Kotlin bei der Entwicklung 
eines Spring Projektes auseinander setzt. Ich werde aus der Perspektive eines Java Entwicklers aufzeigen, welche 
unterschiede bestehen. Der Quellcode des Beispiel Projekts wird auf Github erscheinen jeder Teil der Blogserie wird 
in einem Branch des Beispielprojektes hinterlegt sein.**

Jetzt viel Spass beim lesen.


# Die Dateistruktur
Ich habe auf [start.spring.io](https://start.spring.io/) zwei Konfigurationen ausgewählt einmal mit Java 8 und einmal mit Kotlin.
Dependencies habe ich für den Start leer gelassen, um uns auf das Wesentliche zu konzentrieren.

Der erste Blick auf die Dateistruktur zeigt, dass die Projekte sehr ähnlich aufgebaut sind.



Der es gibt drei kleine Unterschiede:
1. Sourcecodedateien von Java liegen im Unterverzeichnis “Java” und Kotlin im Unterverzeichnis “Kotlin”
2. Die Kotlin Version hat noch ein weiteres Verzeichniss Java um auch Java Dateien ins Projekt einzubinden. 
Dies gelingt, da Kotlin [100% interoperable](https://kotlinlang.org/docs/reference/java-interop.html) zu Java ist. 
3. Die Dateiendungen sind natürlich angepasst. *.java für Java und *.kt für Kotlin Quelldateien.


# Die Hauptdatei OhlalaApplication.java/.kt

Sehen wir uns jetzt die beiden Hauptdateien, der Java und Kotlin implementierung von SpringBoot an.

OhlalaApplication.java:
{% highlight java %}
package com.gjacobsohn.ohlala.ohlala;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class OhlalaApplication {

  public static void main(String[] args) {
     SpringApplication.run(OhlalaApplication.class, args);
  }
{% endhighlight %}

OhlalaApplication.kt:
{% highlight kotlin %}
package com.gjacobsohn.ohlala.ohlala

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class OhlalaApplication

fun main(args: Array<String>) {
  runApplication<OhlalaApplication>(*args)
}
{% endhighlight %}

Bis zur Definition der Klasse sind die Dateien ziemlich ähnlich aber dann kommen ein paar auf den ersten Blick kleine 
aber sehr wichtige unterschiede. Auf die ich hier im Detail eingehen werde.

# Die Deklaration der Methode “main”
Java
{% highlight java %}
public class OhlalaApplication {
   public static void main(String[] args) { ... }
}
{% endhighlight %}

Kotlin
{% highlight kotlin %}
class OhlalaApplication

fun main(args: Array<String>) 
{% endhighlight %}

## Unterschied 1: Die Klassen definiton

Ein kleiner aber sehr wichtiger Unterschied ist, das bei der Kotlin Variante die geschweifte Klammern fehlen.
In Kotlin können diese weg gelassen werden, wenn [die Klasse ohne Body definert wird](https://kotlinlang.org/docs/reference/classes.html). 
Während in Java die Klasse eine Methode besitzt.

## Unterschied 2: Definiton der Funktion main 
Aus dem ersten Unterschied folgt, das die Funktion `main` bei der Java variante innerhalb der Klasse OhlalaApplication definiert ist,
während bei Kotlin die [Definition auf Paketebene](https://kotlinlang.org/docs/reference/visibility-modifiers.html#packages) erfolgt.
Diese Art der Funktionen werden als top-level Functions bezeichnet. 

Daraus resultieren ein paar weiter definitionen:

### Sichtbarkeit
In [Java ist die standard Sichtbarkeit von Methoden](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html) ist `package-private`. 
Zugriffe sind nur innerhalb des Packages erlaubt aber nicht von aussen, um dies zu ändern muss das Keyword `public` genutzt werden.

In Kotlin ist die standard Sichtbarkeit bei top-level Functions `public` und muss so mit nicht angegeben werden.

### Static
In Java wurde das Keyword `static` genutzt um die main funktion aufrufbar zumachn

Wie wir schon oben gesehen haben, ist die Methode `main` in Kotlin ausserhalb einer Klasse definiert.
Da sie nicht innerhalb eines zu instanzierenden Objektes besteht ist die Methode immer `static.

### Rückgabewert
Java folgt dem 90er Jahre paradigma, dass alles explizit angeben wird auch unwichtige Informationen. 
Das führte zu einer sehr ausladenene Syntaxs, ein ehemaliger Professor von mir nannte sie Barock.

Kotlin folgt einem neuen Ansatz der besagt nur wichitge Informationen sollten angeben werden um den Fokus aus wichtige Informatioen zu leiten.
Und vieles wird per definition abgehandelt.

Java: Explizite angabe das keine Rückgabe erfolgt über `void`.
Kotlin: Implizit, da kein Rückgabe wert angeben wurde wird auch keiner zurückgeben. Nach dem Motto, “Wenn nichts angeben wurde folgt auch nichts.”


# Die main Methode


Java {% highlight java %}

SpringApplication.run(OhlalaApplication.class, args);
{% endhighlight %}

In Java sehen wir, das eine statische Methode ausgeführt wird deren Parameter die Klasse OhlalaApplication ist und 
die Parameter mit dem das Programm ausgeführt wurde.

Was hier passiert ist, Java nimmt die Klasse und erezugt ein Objekt der Klasse OhlalaApplication innerhalb 
der statischen Methode `run`.


Kotlin 
{% highlight kotlin %}
runApplication<OhlalaApplication>(*args)
{% endhighlight %} 

In Kotlin wird nur die Methode runApplication mit OhlalaApplication als generischem Parameter ausgeführt. 
Um zu verstehen was dort passiert müssen wir uns die definition der Methode runApplication ansehen.

{% highlight kotlin %}
public inline fun <reified T : kotlin.Any> runApplication(vararg args: kotlin.String): org.springframework.context.ConfigurableApplicationContext
{% endhighlight %} 

Die Methode runApplication ist definiert als [`inline`](https://kotlinlang.org/docs/reference/inline-functions.html) mit einem [`reified Generic T`](https://kotlinlang.org/docs/reference/inline-functions.html#reified-type-parameters) der von Typ [Any](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/index.html#kotlin.Any) sein darf. 
`Any` ist in Kotlin das Eqivalent zu [`Object` in Java](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html).
Diese Inline-Methode ist ein Adapter für die SpringBoot.runApplicaton methode um Java spezifische Aufrufe zu kapseln.

## Inline-Functions
Inline Methoden sind lambda Ausdrücke die direkt in den Code kopiert werden. 
Das bedeutet im Kompilat wird der Code direkt rein kopiert und nicht per Funktionsaufruf ausgeführt. 
Das hat den Nachteil, das es im Resultierenden Bytecode doppelungen gibt und er im zweifelsfall größer wird. 
Aber auch den Vorteil, das funktionsaufrufe die Rechenzeit und speicher brauchen vermieden werden. 
Ähnliches finden wir auch bei den Macro s des C-Compliers.

## reified 
Mit dem Schlüsselwort refined können wir ein Generischen Typen als Parameter in der Funktion nutzen.
Die ist nützlich, wenn wir den Typen nicht nur einsetzen wollen, sondern auch nutzen z.B. über Reflextions.

## Implementierung der runApplication Methode
Wenn wir uns nun die Implementierung der Inline-Funktion ansehen, 
sehen wir auch warum ein weiterer Funktionsaufruf nur übermäßiger Overhead wäre.

{% highlight kotlin %}
inline fun <reified T : Any> runApplication(vararg args: String): ConfigurableApplicationContext =
        SpringApplication.run(T::class.java, *args)
{% endhighlight %}
[SpringApplicationExtensions.kt](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/kotlin/org/springframework/boot/SpringApplicationExtensions.kt)

Wie wir sehen, kapselt die Inline-Funktion nur den SpringApplication.run Aufruf um eine integration von Kotlin zuerreichen.

Eine zweite Implementierung der Inline-Function zeigt uns einen init Parameter.
{% highlight kotlin %}
inline fun <reified T : Any> runApplication(vararg args: String, init: SpringApplication.() -> Unit): ConfigurableApplicationContext =
		SpringApplication(T::class.java).apply(init).run(*args)
{% endhighlight %}
[SpringApplicationExtensions.kt](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/kotlin/org/springframework/boot/SpringApplicationExtensions.kt)

Dieser Paramter ist ein Funktionsparameter. Die Funktion ist definiert als 
Funktion auf dem `SpringApplication` Klasse als RÜckgabewert wird `Unit` zurückgeben was in Kotlin nichts bedeutet. 
Mit diesem Parameter können wir Werte im SpringApplication Objekt Programmatisch setzen bevor der Spring Context erstellt wird. 

Beispiel:

{% highlight kotlin %}
runApplication<OhlalaApplication>(*args) {
    if (ABC = XYZ) {
        setHeadless(False);
    }
}
{% endhighlight %} 


Im nächsten abschnitt werden wir uns um die ersten Enties und Repositories kümmern.
