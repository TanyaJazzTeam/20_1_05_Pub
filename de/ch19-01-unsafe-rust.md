## Unsicherer Rost

Für den gesamten Code, den wir bisher besprochen haben, wurden die Speichersicherheitsgarantien von Rust zur Kompilierzeit durchgesetzt. In Rust ist jedoch eine zweite Sprache versteckt, die diese Speichersicherheitsgarantien nicht erzwingt: Sie heißt *unsicheres Rust* und funktioniert genau wie normales Rust, verleiht uns aber zusätzliche Superkräfte.

Unsicherer Rost existiert, weil die statische Analyse von Natur aus konservativ ist. Wenn der Compiler versucht festzustellen, ob der Code die Garantien erfüllt oder nicht, ist es besser, einige gültige Programme abzulehnen, als einige ungültige Programme zu akzeptieren. Obwohl der Code in Ordnung sein *könnte* , wird der Rust-Compiler den Code ablehnen, wenn er nicht genügend Informationen hat, um sicher zu sein. In diesen Fällen können Sie unsicheren Code verwenden, um dem Compiler mitzuteilen: „Vertrauen Sie mir, ich weiß, was ich tue.“ Der Nachteil ist, dass Sie es auf eigenes Risiko verwenden: Wenn Sie unsicheren Code falsch verwenden, können Probleme aufgrund von Speicherausfällen wie z. B. Nullzeiger-Dereferenzierung auftreten.

Ein weiterer Grund, warum Rust ein unsicheres Alter Ego hat, ist, dass die zugrunde liegende Computerhardware von Natur aus unsicher ist. Wenn Rust Sie keine unsicheren Operationen ausführen ließ, konnten Sie bestimmte Aufgaben nicht ausführen. Rust muss es Ihnen ermöglichen, Low-Level-Programmiersysteme zu verwenden, z. B. direkt mit dem Betriebssystem zu interagieren oder sogar Ihr eigenes Betriebssystem zu schreiben. Die Arbeit mit Low-Level-Programmiersystemen ist eines der Ziele der Sprache. Lassen Sie uns untersuchen, was wir mit unsicherem Rust tun können und wie es geht.

### Unsichere Supermächte

Um zu unsicherem Rust zu wechseln, verwenden Sie das Schlüsselwort `unsafe` und starten Sie dann einen neuen Block, der den unsicheren Code enthält. Sie können in unsicherem Rust fünf Aktionen ausführen, die als *unsichere Superkräfte* bezeichnet werden, die Sie in sicherem Rust nicht ausführen können. Diese Superkräfte umfassen die Fähigkeit:

- Dereferenzieren Sie einen rohen Zeiger
- Rufen Sie eine unsichere Funktion oder Methode auf
- Greifen Sie auf eine veränderliche statische Variable zu oder ändern Sie sie
- Implementieren Sie eine unsichere Eigenschaft
- Zugriffsfelder von `union` s

Es ist wichtig zu verstehen, dass `unsafe` den Borrow-Checker nicht ausschaltet oder andere Sicherheitsüberprüfungen von Rust deaktiviert: Wenn Sie eine Referenz in unsicherem Code verwenden, wird sie trotzdem überprüft. Das `unsafe` Schlüsselwort gibt Ihnen nur Zugriff auf diese fünf Funktionen, die dann vom Compiler nicht auf Speichersicherheit überprüft werden. Sie erhalten immer noch ein gewisses Maß an Sicherheit innerhalb eines unsicheren Blocks.

Darüber hinaus bedeutet `unsafe` nicht, dass der Code innerhalb des Blocks unbedingt gefährlich ist oder dass er definitiv Speichersicherheitsprobleme haben wird: Die Absicht ist, dass Sie als Programmierer sicherstellen, dass der Code innerhalb eines `unsafe` Blocks auf gültige Weise auf den Speicher zugreift .

Menschen sind fehlbar und Fehler werden passieren, aber wenn Sie verlangen, dass diese fünf unsicheren Operationen innerhalb von Blöcken liegen, die mit `unsafe` kommentiert sind, wissen Sie, dass alle Fehler im Zusammenhang mit der Speichersicherheit innerhalb eines `unsafe` Blocks liegen müssen. Halten Sie `unsafe` Blöcke klein; Sie werden später dankbar sein, wenn Sie Speicherfehler untersuchen.

Um unsicheren Code so weit wie möglich zu isolieren, ist es am besten, unsicheren Code in eine sichere Abstraktion einzuschließen und eine sichere API bereitzustellen, die wir später in diesem Kapitel besprechen werden, wenn wir unsichere Funktionen und Methoden untersuchen. Teile der Standardbibliothek sind als sichere Abstraktionen über unsicheren Code implementiert, der geprüft wurde. Durch das Einschließen von unsicherem Code in eine sichere Abstraktion wird verhindert, dass die Verwendung von `unsafe` Code an alle Stellen gelangt, an denen Sie oder Ihre Benutzer möglicherweise die mit `unsafe` Code implementierte Funktionalität verwenden möchten, da die Verwendung einer sicheren Abstraktion sicher ist.

Schauen wir uns der Reihe nach jede der fünf unsicheren Supermächte an. Wir werden uns auch einige Abstraktionen ansehen, die eine sichere Schnittstelle zu unsicherem Code bieten.

### Dereferenzieren eines Rohzeigers

In Kapitel 4, in den [„Dangling References“](ch04-02-references-and-borrowing.html#dangling-references)<!-- ignorieren --> Abschnitt haben wir erwähnt, dass der Compiler sicherstellt, dass Verweise immer gültig sind. Unsafe Rust hat zwei neue Typen, sogenannte *Raw-Zeiger* , die Referenzen ähneln. Wie bei Referenzen können Rohzeiger unveränderlich oder veränderlich sein und werden als `*const T` bzw. `*mut T` geschrieben. Das Sternchen ist nicht der Dereferenzierungsoperator; es ist Teil des Typnamens. Im Zusammenhang mit rohen Zeigern bedeutet *unveränderlich* , dass der Zeiger nach der Dereferenzierung nicht direkt zugewiesen werden kann.

Anders als Referenzen und intelligente Zeiger, Rohzeiger:

- Sie dürfen die Ausleihregeln ignorieren, indem sie sowohl unveränderliche als auch veränderliche Zeiger oder mehrere veränderliche Zeiger auf denselben Ort haben
- Zeigen nicht garantiert auf gültigen Speicher
- Dürfen null sein
- Implementieren Sie keine automatische Bereinigung

Indem Sie sich gegen die Durchsetzung dieser Garantien durch Rust entscheiden, können Sie die garantierte Sicherheit im Austausch für eine bessere Leistung oder die Möglichkeit, mit einer anderen Sprache oder Hardware zu kommunizieren, aufgeben, für die die Garantien von Rust nicht gelten.

Listing 19.1 zeigt, wie man aus Referenzen einen unveränderlichen und einen veränderlichen Rohzeiger erzeugt.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-01/src/main.rs:here}}
```

<span class="caption">Listing 19-1: Rohzeiger aus Referenzen erzeugen</span>

Beachten Sie, dass wir das `unsafe` Schlüsselwort nicht in diesen Code aufnehmen. Wir können rohe Zeiger in sicherem Code erstellen; Wir können rohe Zeiger einfach nicht außerhalb eines unsicheren Blocks dereferenzieren, wie Sie gleich sehen werden.

Wir haben Rohzeiger erstellt, indem wir `as` verwendet haben, um eine unveränderliche und eine veränderliche Referenz in ihre entsprechenden Rohzeigertypen umzuwandeln. Da wir sie direkt aus Referenzen erstellt haben, die garantiert gültig sind, wissen wir, dass diese speziellen Rohzeiger gültig sind, aber wir können diese Annahme nicht für jeden Rohzeiger treffen.

Als Nächstes erstellen wir einen Rohzeiger, dessen Gültigkeit wir nicht beurteilen können. Listing 19-2 zeigt, wie man einen rohen Zeiger auf eine beliebige Stelle im Speicher erzeugt. Der Versuch, beliebigen Speicher zu verwenden, ist undefiniert: An dieser Adresse können Daten vorhanden sein oder auch nicht, der Compiler optimiert den Code möglicherweise so, dass kein Speicherzugriff erfolgt, oder das Programm weist möglicherweise einen Segmentierungsfehler auf. Normalerweise gibt es keinen guten Grund, solchen Code zu schreiben, aber es ist möglich.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-02/src/main.rs:here}}
```

<span class="caption">Listing 19-2: Erstellen eines rohen Zeigers auf eine beliebige Speicheradresse</span>

Denken Sie daran, dass wir Rohzeiger in sicherem Code erstellen können, aber wir können Rohzeiger nicht *dereferenzieren* und die Daten lesen, auf die gezeigt wird. In Listing 19.3 verwenden wir den Dereferenzierungsoperator `*` für einen Rohzeiger, der einen `unsafe` Block erfordert.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-03/src/main.rs:here}}
```

<span class="caption">Listing 19-3: Dereferenzieren von rohen Zeigern innerhalb eines <code>unsafe</code> Blocks</span>

Das Erstellen eines Zeigers schadet nicht; Nur wenn wir versuchen, auf den Wert zuzugreifen, auf den er zeigt, könnten wir es mit einem ungültigen Wert zu tun haben.

Beachten Sie auch, dass wir in Listing 19-1 und 19-3 `*const i32` und `*mut i32` -Rohzeiger erstellt haben, die beide auf dieselbe Speicherstelle zeigten, an der `num` gespeichert ist. Wenn wir stattdessen versucht hätten, eine unveränderliche und eine veränderliche Referenz auf `num` zu erstellen, wäre der Code nicht kompiliert worden, da die Besitzregeln von Rust keine veränderliche Referenz gleichzeitig mit unveränderlichen Referenzen zulassen. Mit rohen Zeigern können wir einen veränderlichen Zeiger und einen unveränderlichen Zeiger auf denselben Ort erstellen und Daten durch den veränderlichen Zeiger ändern, wodurch möglicherweise ein Datenrennen entsteht. Vorsichtig sein!

Warum sollten Sie bei all diesen Gefahren jemals rohe Zeiger verwenden? Ein wichtiger Anwendungsfall ist die Schnittstelle mit C-Code, wie Sie im nächsten Abschnitt [„Aufrufen einer unsicheren Funktion oder Methode“ sehen werden.](#calling-an-unsafe-function-or-method)<!-- ignorieren --> Ein anderer Fall ist der Aufbau sicherer Abstraktionen, die der Borrow-Checker nicht versteht. Wir führen unsichere Funktionen ein und sehen uns dann ein Beispiel für eine sichere Abstraktion an, die unsicheren Code verwendet.

### Aufrufen einer unsicheren Funktion oder Methode

Der zweite Operationstyp, der einen unsicheren Block erfordert, sind Aufrufe unsicherer Funktionen. Unsichere Funktionen und Methoden sehen genauso aus wie normale Funktionen und Methoden, aber sie haben vor dem Rest der Definition ein zusätzliches `unsafe` Zeichen. Das Schlüsselwort `unsafe` in diesem Zusammenhang zeigt an, dass die Funktion Anforderungen hat, die wir einhalten müssen, wenn wir diese Funktion aufrufen, da Rust nicht garantieren kann, dass wir diese Anforderungen erfüllt haben. Indem wir eine unsichere Funktion innerhalb eines `unsafe` Blocks aufrufen, sagen wir, dass wir die Dokumentation dieser Funktion gelesen haben und die Verantwortung für die Einhaltung der Verträge der Funktion übernehmen.

Hier ist eine unsichere Funktion namens " `dangerous` ", die nichts in ihrem Körper tut:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

Wir müssen die `dangerous` Funktion innerhalb eines separaten `unsafe` Blocks aufrufen. Wenn wir versuchen, „ `dangerous` “ ohne den `unsafe` Block aufzurufen, erhalten wir eine Fehlermeldung:

```console
{{#include ../listings/ch19-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

Indem wir den `unsafe` Block um unseren Aufruf von `dangerous` einfügen, versichern wir Rust, dass wir die Dokumentation der Funktion gelesen haben, dass wir verstehen, wie man sie richtig verwendet, und dass wir verifiziert haben, dass wir den Vertrag der Funktion erfüllen.

Körper von unsicheren Funktionen sind effektiv `unsafe` Blöcke. Um also andere unsichere Operationen innerhalb einer unsicheren Funktion auszuführen, müssen wir keinen weiteren `unsafe` Block hinzufügen.

#### Erstellen einer sicheren Abstraktion über unsicheren Code

Nur weil eine Funktion unsicheren Code enthält, heißt das nicht, dass wir die gesamte Funktion als unsicher markieren müssen. Tatsächlich ist das Verpacken von unsicherem Code in eine sichere Funktion eine gängige Abstraktion. Lassen Sie uns als Beispiel eine Funktion aus der Standardbibliothek, `split_at_mut` , untersuchen, die unsicheren Code erfordert, und untersuchen, wie wir ihn implementieren könnten. Diese sichere Methode ist für veränderliche Slices definiert: Sie nimmt ein Slice und macht daraus zwei, indem das Slice an dem als Argument angegebenen Index geteilt wird. Listing 19.4 zeigt, wie `split_at_mut` verwendet wird.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-04/src/main.rs:here}}
```

<span class="caption">Listing 19-4: Verwendung der sicheren <code>split_at_mut</code> Funktion</span>

Wir können diese Funktion nicht nur mit sicherem Rust implementieren. Ein Versuch könnte etwa so aussehen wie in Listing 19.5, das nicht kompiliert wird. Der Einfachheit halber implementieren wir `split_at_mut` als Funktion und nicht als Methode und nur für Slices von `i32` Werten und nicht für einen generischen Typ `T`

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-05/src/main.rs:here}}
```

<span class="caption">Listing 19-5: Eine versuchte Implementierung von <code>split_at_mut</code> nur mit sicherem Rust</span>

Diese Funktion ruft zuerst die Gesamtlänge des Slice ab. Dann wird behauptet, dass der als Parameter angegebene Index innerhalb des Slice liegt, indem geprüft wird, ob er kleiner oder gleich der Länge ist. Die Assertion bedeutet, dass, wenn wir einen Index übergeben, der größer ist als die Länge, an der der Slice aufgeteilt werden soll, die Funktion in Panik gerät, bevor sie versucht, diesen Index zu verwenden.

Dann geben wir zwei änderbare Slices in einem Tupel zurück: eines vom Anfang des ursprünglichen Slice bis zum `mid` Index und ein weiteres von der `mid` bis zum Ende des Slice.

Wenn wir versuchen, den Code in Listing 19.5 zu kompilieren, erhalten wir eine Fehlermeldung.

```console
{{#include ../listings/ch19-advanced-features/listing-19-05/output.txt}}
```

Der Borrow-Checker von Rust kann nicht verstehen, dass wir verschiedene Teile des Slice ausleihen; es weiß nur, dass wir zweimal von demselben Slice borgen. Das Ausleihen verschiedener Teile eines Slice ist grundsätzlich in Ordnung, da sich die beiden Slices nicht überlappen, aber Rust ist nicht schlau genug, dies zu wissen. Wenn wir wissen, dass der Code in Ordnung ist, aber Rust nicht, ist es an der Zeit, nach unsicherem Code zu greifen.

Listing 19.6 zeigt, wie Sie einen `unsafe` Block, einen rohen Zeiger und einige Aufrufe unsicherer Funktionen verwenden, damit die Implementierung von `split_at_mut` funktioniert.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-06/src/main.rs:here}}
```

<span class="caption">Listing 19-6: Verwendung von unsicherem Code in der Implementierung der <code>split_at_mut</code> Funktion</span>

Rückruf von [„The Slice Type“]<!-- ignorieren --> Abschnitt in Kapitel 4, dass Slices ein Zeiger auf einige Daten und die Länge des Slice sind. Wir verwenden die `len` -Methode, um die Länge eines Slice zu erhalten, und die `as_mut_ptr` Methode, um auf den Rohzeiger eines Slice zuzugreifen. Da wir in diesem Fall einen änderbaren Slice zu `i32` Werten haben, gibt `as_mut_ptr` einen rohen Zeiger mit dem Typ `*mut i32` , den wir in der Variablen `ptr` gespeichert haben.

Wir behalten die Behauptung bei, dass der `mid` Index innerhalb des Slice liegt. Dann kommen wir zum unsicheren Code: Die Funktion `slice::from_raw_parts_mut` nimmt einen rohen Zeiger und eine Länge und erstellt ein Slice. Wir verwenden diese Funktion, um ein Slice zu erstellen, das bei `ptr` beginnt und `mid` ist. Dann rufen wir die Methode `add` für `ptr` mit `mid` als Argument auf, um einen rohen Zeiger zu erhalten, der bei `mid` beginnt, und wir erstellen einen Slice mit diesem Zeiger und der verbleibenden Anzahl von Elementen nach `mid` als Länge.

Die `slice::from_raw_parts_mut` ist unsicher, da sie einen rohen Zeiger akzeptiert und darauf vertrauen muss, dass dieser Zeiger gültig ist. Die `add` Methode für Rohzeiger ist ebenfalls unsicher, da sie darauf vertrauen muss, dass die Offset-Position auch ein gültiger Zeiger ist. Daher mussten wir einen `unsafe` Block um unsere Aufrufe von `slice::from_raw_parts_mut` und `add` setzen, damit wir sie aufrufen konnten. Wenn wir uns den Code ansehen und die Behauptung hinzufügen, dass `mid` kleiner oder gleich `len` sein muss, können wir sagen, dass alle rohen Zeiger, die innerhalb des `unsafe` Blocks verwendet werden, gültige Zeiger auf Daten innerhalb des Slice sind. Dies ist eine akzeptable und angemessene Verwendung von `unsafe` .

Beachten Sie, dass wir die resultierende Funktion `split_at_mut` nicht als `unsafe` markieren müssen und diese Funktion von sicherem Rust aus aufrufen können. Wir haben eine sichere Abstraktion zum unsicheren Code mit einer Implementierung der Funktion erstellt, die `unsafe` Code auf sichere Weise verwendet, da sie nur gültige Zeiger aus den Daten erstellt, auf die diese Funktion Zugriff hat.

Im Gegensatz dazu würde die Verwendung von `slice::from_raw_parts_mut` in Listing 19.7 wahrscheinlich abstürzen, wenn das Slice verwendet wird. Dieser Code nimmt einen beliebigen Speicherort und erstellt einen Abschnitt mit einer Länge von 10.000 Elementen.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-07/src/main.rs:here}}
```

<span class="caption">Listing 19-7: Erstellen eines Slice aus einem beliebigen Speicherplatz</span>

Wir besitzen den Speicher an diesem willkürlichen Ort nicht, und es gibt keine Garantie dafür, dass der von diesem Code erstellte Slice gültige `i32` Werte enthält. Der Versuch, `values` so zu verwenden, als ob es sich um einen gültigen Slice handelt, führt zu undefiniertem Verhalten.

#### Verwenden `extern` Funktionen zum Aufrufen von externem Code

Manchmal muss Ihr Rust-Code möglicherweise mit Code interagieren, der in einer anderen Sprache geschrieben wurde. Rust hat dafür ein Schlüsselwort, `extern` , das die Erstellung und Verwendung eines *Foreign Function Interface (FFI)* erleichtert. Ein FFI ist eine Möglichkeit für eine Programmiersprache, Funktionen zu definieren und es einer anderen (fremden) Programmiersprache zu ermöglichen, diese Funktionen aufzurufen.

Listing 19.8 zeigt, wie Sie eine Integration mit der `abs` -Funktion aus der C-Standardbibliothek einrichten. Funktionen, die in `extern` Blöcken deklariert sind, können immer nicht sicher aus Rust-Code aufgerufen werden. Der Grund dafür ist, dass andere Sprachen die Regeln und Garantien von Rust nicht durchsetzen und Rust sie nicht überprüfen kann, sodass die Verantwortung für die Sicherheit beim Programmierer liegt.

<span class="filename">Dateiname: src / main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-08/src/main.rs}}
```

<span class="caption">Listing 19-8: Deklarieren und Aufrufen einer <code>extern</code> Funktion, die in einer anderen Sprache definiert ist</span>

Innerhalb des externen `extern "C"` -Blocks listen wir die Namen und Signaturen externer Funktionen aus einer anderen Sprache auf, die wir aufrufen möchten. Der `"C"` -Teil definiert, welche *Application Binary Interface (ABI)* die externe Funktion verwendet: Die ABI definiert, wie die Funktion auf Assemblyebene aufgerufen wird. Die ABI `"C"` ist die gebräuchlichste und folgt der ABI der Programmiersprache C.

> #### Rust-Funktionen aus anderen Sprachen aufrufen
>
> Wir können auch `extern` verwenden, um eine Schnittstelle zu erstellen, die es anderen Sprachen ermöglicht, Rust-Funktionen aufzurufen. Anstelle eines `extern` Blocks fügen wir das Schlüsselwort `extern` hinzu und geben die zu verwendende ABI direkt vor dem Schlüsselwort `fn` an. Wir müssen auch eine `#[no_mangle]` hinzufügen, um den Rust-Compiler anzuweisen, den Namen dieser Funktion nicht zu manipulieren. *Mangling* ist, wenn ein Compiler den Namen, den wir einer Funktion gegeben haben, in einen anderen Namen ändert, der mehr Informationen für andere Teile des Kompilierungsprozesses enthält, aber weniger lesbar ist. Jeder Programmiersprachen-Compiler verstümmelt Namen etwas anders, damit eine Rust-Funktion von anderen Sprachen benennbar ist, müssen wir die Namensverzerrung des Rust-Compilers deaktivieren.
>
> Im folgenden Beispiel machen wir die Funktion `call_from_c` über C-Code zugänglich, nachdem sie in eine gemeinsam genutzte Bibliothek kompiliert und von C gelinkt wurde:
>
> ```rust
> #[no_mangle]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Rust function from C!");
> }
> ```
>
> Diese Verwendung von `extern` erfordert kein `unsafe` .

### Zugreifen auf oder Ändern einer veränderlichen statischen Variablen

Bis jetzt haben wir nicht über *globale Variablen* gesprochen, die Rust zwar unterstützt, aber mit Rusts Eigentumsregeln problematisch sein kann. Wenn zwei Threads auf dieselbe veränderliche globale Variable zugreifen, kann dies zu einem Datenrennen führen.

In Rust werden globale Variablen als *statische* Variablen bezeichnet. Listing 19.9 zeigt ein Beispiel für die Deklaration und Verwendung einer statischen Variablen mit einem String-Slice als Wert.

<span class="filename">Dateiname: src / main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-09/src/main.rs}}
```

<span class="caption">Listing 19.9: Definition und Verwendung einer unveränderlichen statischen Variablen</span>

Statische Variablen ähneln Konstanten, die wir im [Abschnitt „Unterschiede zwischen Variablen und Konstanten“](ch03-01-variables-and-mutability.html#constants) besprochen haben.<!-- ignorieren --> Abschnitt in Kapitel 3. Die Namen von statischen Variablen stehen per Konvention in `SCREAMING_SNAKE_CASE` . Statische Variablen können nur Referenzen mit der `'static` Lebensdauer speichern, was bedeutet, dass der Rust-Compiler die Lebensdauer ermitteln kann und wir sie nicht explizit annotieren müssen. Der Zugriff auf eine unveränderliche statische Variable ist sicher.

Konstanten und unveränderliche statische Variablen mögen ähnlich erscheinen, aber ein subtiler Unterschied besteht darin, dass Werte in einer statischen Variablen eine feste Adresse im Speicher haben. Die Verwendung des Werts greift immer auf dieselben Daten zu. Konstanten hingegen dürfen ihre Daten duplizieren, wann immer sie verwendet werden.

Ein weiterer Unterschied zwischen Konstanten und statischen Variablen besteht darin, dass statische Variablen veränderlich sein können. Der Zugriff auf und die Änderung veränderlicher statischer Variablen ist *unsicher* . Listing 19.10 zeigt, wie man eine veränderliche statische Variable namens `COUNTER` deklariert, auf sie zugreift und sie ändert.

<span class="filename">Dateiname: src / main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-10/src/main.rs}}
```

<span class="caption">Listing 19-10: Das Lesen oder Schreiben in eine änderbare statische Variable ist unsicher</span>

Wie bei regulären Variablen spezifizieren wir die Veränderlichkeit mit dem Schlüsselwort `mut` . Jeder Code, der von `COUNTER` liest oder schreibt, muss sich in einem `unsafe` Block befinden. Dieser Code kompiliert und `COUNTER: 3` aus, wie wir es erwarten würden, da es sich um einen Single-Thread handelt. Der Zugriff mehrerer Threads auf `COUNTER` würde wahrscheinlich zu Datenrennen führen.

Bei änderbaren Daten, die global zugänglich sind, ist es schwierig sicherzustellen, dass es keine Datenrennen gibt, weshalb Rust änderbare statische Variablen als unsicher ansieht. Wenn möglich, ist es vorzuziehen, die Parallelitätstechniken und Thread-sicheren intelligenten Zeiger zu verwenden, die wir in Kapitel 16 besprochen haben, damit der Compiler überprüft, ob der Zugriff auf Daten von verschiedenen Threads sicher erfolgt.

### Implementieren einer unsicheren Eigenschaft

Ein weiterer Anwendungsfall für `unsafe` ist die Implementierung einer unsicheren Eigenschaft. Eine Eigenschaft ist unsicher, wenn mindestens eine ihrer Methoden eine Invariante hat, die der Compiler nicht überprüfen kann. Wir können eine Eigenschaft als `unsafe` deklarieren, indem wir das Schlüsselwort `unsafe` vor `trait` einfügen und die Implementierung der Eigenschaft ebenfalls als `unsafe` markieren, wie in Listing 19.11 gezeigt.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-11/src/main.rs}}
```

<span class="caption">Listing 19-11: Definieren und Implementieren eines unsicheren Merkmals</span>

Durch die Verwendung von `unsafe impl` versprechen wir, dass wir die Invarianten aufrechterhalten, die der Compiler nicht überprüfen kann.

Erinnern Sie sich als Beispiel an die Marker-Merkmale „ `Sync` “ und „ `Send` “, die wir im <a href="ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits" data-md-type="link">Abschnitt „Erweiterbare Parallelität mit den Merkmalen „ `Sync` “ und „ `Send` “</a> besprochen haben.<!-- ignorieren --> Abschnitt in Kapitel 16: Der Compiler implementiert diese Eigenschaften automatisch, wenn unsere Typen ausschließlich aus `Send` und `Sync` -Typen bestehen. Wenn wir einen Typ implementieren, der einen Typ enthält, der nicht `Send` oder `Sync` ist, z. B. rohe Zeiger, und wir diesen Typ als `Send` oder `Sync` markieren möchten, müssen wir `unsafe` verwenden. Rust kann nicht überprüfen, ob unser Typ die Garantien erfüllt, dass er sicher über Threads gesendet oder von mehreren Threads aus aufgerufen werden kann; Daher müssen wir diese Überprüfungen manuell durchführen und als solche mit `unsafe` .

### Zugriff auf Felder einer Union

Die letzte Aktion, die nur mit `unsafe` funktioniert, ist der Zugriff auf Felder einer *union* . Eine `union` ähnelt einem `struct` , aber es wird jeweils nur ein deklariertes Feld in einer bestimmten Instanz verwendet. Unions werden hauptsächlich als Schnittstelle zu Unions in C-Code verwendet. Der Zugriff auf Union-Felder ist unsicher, da Rust nicht garantieren kann, welche Art von Daten derzeit in der Union-Instanz gespeichert werden. Mehr über Unions erfahren Sie in [der Rust-Referenz] .

### Wann sollte unsicherer Code verwendet werden?

`unsafe` zu verwenden, um eine der fünf gerade besprochenen Aktionen (Superkräfte) auszuführen, ist nicht falsch oder sogar verpönt. Aber es ist schwieriger, `unsafe` Code zu korrigieren, da der Compiler nicht helfen kann, die Speichersicherheit aufrechtzuerhalten. Wenn Sie einen Grund haben, `unsafe` Code zu verwenden, können Sie dies tun, und die explizite `unsafe` Anmerkung macht es einfacher, die Ursache von Problemen aufzuspüren, wenn sie auftreten.


[„The Slice Type“]: ch04-03-slices.html#the-slice-type
[der Rust-Referenz]: ../reference/items/unions.html