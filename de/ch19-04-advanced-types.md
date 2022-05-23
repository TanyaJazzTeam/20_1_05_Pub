## Fortgeschrittene Typen

Das Rust-Typsystem hat einige Funktionen, die wir in diesem Buch erwähnt, aber noch nicht besprochen haben. Wir beginnen damit, Newtypes im Allgemeinen zu diskutieren, während wir untersuchen, warum Newtypes als Typen nützlich sind. Dann wenden wir uns Typ-Aliassen zu, einem Feature, das newtypes ähnelt, aber eine etwas andere Semantik hat. Wir besprechen auch die `!` Typ und Typen mit dynamischer Größe.

### Verwenden des Newtype-Musters für Typsicherheit und Abstraktion

> Hinweis: In diesem Abschnitt wird davon ausgegangen, dass Sie den vorherigen Abschnitt [„Verwenden des Newtype-Musters zum Implementieren externer Merkmale für externe Typen“ gelesen haben.]<!-- ignorieren -->

Das newtype-Muster ist nützlich für Aufgaben, die über die bisher besprochenen hinausgehen, einschließlich der statischen Durchsetzung, dass Werte niemals verwechselt werden, und der Angabe der Einheiten eines Werts. Sie haben in Listing 19-15 ein Beispiel für die Verwendung von Newtypes zur Angabe von Einheiten gesehen: Denken Sie daran, dass die Strukturen `Millimeters` und `Meters` `u32` Werte in einen Newtype verpackt haben. Wenn wir eine Funktion mit einem Parameter vom Typ `Millimeters` geschrieben hätten, könnten wir kein Programm kompilieren, das versehentlich versucht hat, diese Funktion mit einem Wert vom Typ `Meters` oder einem einfachen `u32` .

Eine weitere Verwendung des Newtype-Musters besteht darin, einige Implementierungsdetails eines Typs zu abstrahieren: Der neue Typ kann eine öffentliche API verfügbar machen, die sich von der API des privaten inneren Typs unterscheidet.

Newtypes können auch interne Implementierungen verbergen. Beispielsweise könnten wir einen `People` bereitstellen, um eine `HashMap<i32, String>` zu umschließen, die die mit ihrem Namen verknüpfte ID einer Person speichert. Code, der `People` verwendet, würde nur mit der von uns bereitgestellten öffentlichen API interagieren, z. B. eine Methode zum Hinzufügen einer Namenszeichenfolge zur `People` -Sammlung; Dieser Code müsste nicht wissen, dass wir Namen intern eine `i32` -ID zuweisen. Das Newtype-Muster ist eine einfache Möglichkeit, eine Kapselung zu erreichen, um Implementierungsdetails zu verbergen, die wir in [„Kapselung, die Implementierungsdetails](ch17-01-what-is-oo.html#encapsulation-that-hides-implementation-details) verbirgt“ besprochen haben.<!-- ignorieren --> Abschnitt von Kapitel 17.

### Erstellen von Typsynonymen mit Typaliasen

Zusammen mit dem Newtype-Muster bietet Rust die Möglichkeit, einen *Typ-Alias* zu deklarieren, um einem vorhandenen Typ einen anderen Namen zu geben. Dazu verwenden wir das Schlüsselwort `type` . Zum Beispiel können wir den Alias `Kilometers` zu `i32` wie folgt erstellen:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

Nun ist der Alias `Kilometers` ein *Synonym* für `i32` ; Im Gegensatz zu den Typen `Millimeters` und `Meters` , die wir in Listing 19.15 erstellt haben, ist `Kilometers` kein separater, neuer Typ. Werte vom Typ `Kilometers` werden wie Werte vom Typ `i32` behandelt:

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

Da `Kilometers` und `i32` vom gleichen Typ sind, können wir Werte beider Typen hinzufügen und `Kilometers` an Funktionen übergeben, die `i32` Parameter verwenden. Mit dieser Methode erhalten wir jedoch nicht die Art von Überprüfungsvorteilen, die wir aus dem zuvor besprochenen Newtype-Muster ziehen.

Der Hauptanwendungsfall für Typsynonyme besteht darin, Wiederholungen zu reduzieren. Zum Beispiel könnten wir einen langen Typ wie diesen haben:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Das Schreiben dieses langen Typs in Funktionssignaturen und als Typanmerkungen im gesamten Code kann mühsam und fehleranfällig sein. Stellen Sie sich vor, Sie hätten ein Projekt voller Code wie das in Listing 19-24.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-24/src/main.rs:here}}
```

<span class="caption">Listing 19-24: Verwendung eines langen Typs an vielen Stellen</span>

Ein Typ-Alias macht diesen Code besser handhabbar, indem er die Wiederholung reduziert. In Listing 19.25 haben wir einen Alias namens `Thunk` für den verbose-Typ eingeführt und können alle Verwendungen des Typs durch den kürzeren Alias `Thunk` ersetzen.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-25/src/main.rs:here}}
```

<span class="caption">Listing 19–25: Einführung eines Typenalias <code>Thunk</code> zur Reduzierung von Wiederholungen</span>

Dieser Code ist viel einfacher zu lesen und zu schreiben! Die Wahl eines aussagekräftigen Namens für einen Typ-Alias kann auch dabei helfen, Ihre Absicht zu kommunizieren ( *thunk* ist ein Wort für Code, der zu einem späteren Zeitpunkt ausgewertet werden soll, also ist es ein angemessener Name für eine Closure, die gespeichert wird).

Typaliase werden auch häufig mit dem `Result<T, E>` verwendet, um Wiederholungen zu reduzieren. Betrachten Sie das Modul `std::io` in der Standardbibliothek. E/A-Operationen geben oft ein `Result<T, E>` zurück, um Situationen zu behandeln, in denen Operationen nicht funktionieren. Diese Bibliothek hat eine `std::io::Error` -Struktur, die alle möglichen I/O-Fehler darstellt. Viele der Funktionen in `std::io` geben das `Result<T, E>` zurück, wobei das `E` `std::io::Error` ist, wie z. B. diese Funktionen im `Write` Trait:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

Das `Result<..., Error>` wird häufig wiederholt. Als solches hat `std::io` diese Typ-Alias-Deklaration:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

Da sich diese Deklaration im `std::io` -Modul befindet, können wir den vollständig qualifizierten Alias `std::io::Result<T>` verwenden – das heißt, ein `Result<T, E>` bei dem das `E` als `std::io::Error` ausgefüllt ist `std::io::Error` . Die Signaturen der `Write` Trait-Funktion sehen am Ende so aus:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

Der Typ-Alias hilft auf zweierlei Weise: Er erleichtert das Schreiben von Code *und* gibt uns eine konsistente Schnittstelle für alle `std::io` . Da es sich um einen Alias handelt, ist es nur ein weiteres `Result<T, E>` , was bedeutet, dass wir alle Methoden verwenden können, die mit `Result<T, E>` arbeiten, sowie eine spezielle Syntax wie das `?` Operator.

### Der Nie-Typ, der nie zurückkehrt

Rust hat einen speziellen Typ namens `!` das ist in der Typentheorie als *leerer Typ* bekannt, weil es keine Werte hat. Wir nennen ihn lieber *never type* , weil er anstelle des Rückgabetyps steht, wenn eine Funktion niemals zurückkehrt. Hier ist ein Beispiel:

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

Dieser Code wird gelesen als „die Funktionsleiste `bar` nie zurück“. Funktionen, die nie zurückgeben, werden *divergierende Funktionen* genannt. Wir können keine Werte des Typs erstellen `!` `bar` kann also niemals zurückkehren.

Aber was nützt ein Typ, für den man niemals Werte schaffen kann? Rufen Sie den Code aus Listing 2-5 auf; wir haben einen Teil davon hier in Listing 19-26 reproduziert.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

<span class="caption">Listing 19-26: Ein <code>match</code> mit einem Arm, der auf <code>continue</code> endet</span>

Damals haben wir einige Details in diesem Code übersprungen. In Kapitel 6 in [„Der `match` -Control-Flow-Operator“](ch06-02-match.html#the-match-control-flow-operator)<!-- ignorieren --> Abschnitt haben wir besprochen, dass `match` -Waffen alle den gleichen Typ zurückgeben müssen. So funktioniert zum Beispiel der folgende Code nicht:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

Der Typ der `guess` in diesem Code müsste eine Ganzzahl *und* ein String sein, und Rust verlangt, dass diese `guess` nur einen Typ hat. Also, was wird `continue` ? Wie konnten wir `continue` Listing 19-26 ein `u32` von einem Arm zurückgeben und einen anderen Arm haben, der mit Continue endet?

Wie Sie vielleicht erraten haben, hat `continue` ein `!` Wert. Das heißt, wenn Rust die Art der `guess` berechnet, betrachtet es beide Match-Arme, ersteres mit einem Wert von `u32` und letzteres mit einem `!` Wert. Weil `!` niemals einen Wert haben kann, entscheidet Rust, dass die Art der `guess` `u32` ist.

Formal lässt sich dieses Verhalten so beschreiben, dass Ausdrücke vom Typ `!` kann in jeden anderen Typ gezwungen werden. Wir dürfen diesen `match` -Arm mit `continue` beenden, weil `continue` keinen Wert zurückgibt; Stattdessen verschiebt es die Steuerung zurück an den Anfang der Schleife, sodass wir im `Err` -Fall nie einen Wert zuweisen, um zu `guess` .

Der Nie-Typ ist nützlich bei der `panic!` auch Makro. Erinnern Sie sich an die `unwrap` Funktion, die wir für `Option<T>` -Werte aufrufen, um einen Wert oder eine Panik zu erzeugen? Hier ist seine Definition:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

In diesem Code passiert das Gleiche wie im `match` in Listing 19-26: Rust sieht, dass `val` den Typ `T` hat und `panic!` hat den typ `!` , also ist das Ergebnis des gesamten `match` `T` Dieser Code funktioniert, weil `panic!` erzeugt keinen Wert; es beendet das Programm. Im `None` -Fall geben wir keinen Wert von `unwrap` , daher ist dieser Code gültig.

Ein letzter Ausdruck, der den Typ `!` ist eine `loop` :

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

Hier endet die Schleife nie, also `!` ist der Wert des Ausdrucks. Dies wäre jedoch nicht der Fall, wenn wir ein `break` einschließen würden, da die Schleife enden würde, wenn sie zum `break` käme.

### Typen mit dynamischer Größe und das `Sized`

Da Rust bestimmte Details kennen muss, wie z. B. wie viel Platz für einen Wert eines bestimmten Typs zugewiesen werden soll, gibt es eine Ecke seines Typsystems, die verwirrend sein kann: das Konzept der *Typen mit dynamischer Größe* . Diese Typen, die manchmal als *DSTs* oder nicht *angepasste Typen* bezeichnet werden, ermöglichen es uns, Code mit Werten zu schreiben, deren Größe wir nur zur Laufzeit kennen können.

Sehen wir uns die Details eines Typs mit dynamischer Größe namens `str` an, den wir im gesamten Buch verwendet haben. Richtig, nicht `&str` , sondern `str` allein ist eine DST. Wir können bis zur Laufzeit nicht wissen, wie lang der String ist, was bedeutet, dass wir weder eine Variable vom Typ `str` erstellen noch ein Argument vom Typ `str` nehmen können. Betrachten Sie den folgenden Code, der nicht funktioniert:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust muss wissen, wie viel Speicher für einen beliebigen Wert eines bestimmten Typs zugewiesen werden soll, und alle Werte eines Typs müssen dieselbe Menge an Speicher verwenden. Wenn Rust uns erlauben würde, diesen Code zu schreiben, müssten diese beiden `str` -Werte den gleichen Platz einnehmen. Aber sie haben unterschiedliche Längen: `s1` benötigt 12 Bytes Speicherplatz und `s2` benötigt 15. Aus diesem Grund ist es nicht möglich, eine Variable zu erstellen, die einen Typ mit dynamischer Größe enthält.

Also, was machen wir? In diesem Fall kennen Sie die Antwort bereits: Wir machen die Typen von `s1` und `s2` zu a `&str` statt a `str` . Erinnern Sie sich daran, dass in den [„String Slices“]<!-- ignorieren --> Abschnitt von Kapitel 4 haben wir gesagt, dass die Slice-Datenstruktur die Startposition und die Länge des Slice speichert.

Obwohl also a `&T` ein einzelner Wert ist, der die Speicheradresse speichert, wo sich das `T` befindet, sind a `&str` *zwei* Werte: die Adresse des `str` und seine Länge. Daher können wir die Größe eines `&str` -Werts zur Kompilierzeit kennen: Er ist doppelt so lang wie ein `usize` . Das heißt, wir kennen immer die Größe von a `&str` , egal wie lang die Zeichenfolge ist, auf die es sich bezieht. Im Allgemeinen werden Typen mit dynamischer Größe in Rust auf diese Weise verwendet: Sie haben ein zusätzliches Bit an Metadaten, das die Größe der dynamischen Informationen speichert. Die goldene Regel von Typen mit dynamischer Größe lautet, dass wir Werte von Typen mit dynamischer Größe immer hinter einen Zeiger setzen müssen.

Wir können `str` mit allen Arten von Zeigern kombinieren: zum Beispiel `Box<str>` oder `Rc<str>` . Tatsächlich haben Sie das schon einmal gesehen, aber mit einem anderen Typ mit dynamischer Größe: Eigenschaften. Jedes Merkmal ist ein Typ mit dynamischer Größe, auf den wir uns beziehen können, indem wir den Namen des Merkmals verwenden. In Kapitel 17 in der [„Verwendung von Eigenschaftsobjekten, die Werte verschiedener Typen zulassen“](ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types)<!-- ignorieren --> Abschnitt haben wir erwähnt, dass wir Eigenschaften hinter einen Zeiger setzen müssen, um Eigenschaften als Eigenschaftsobjekte zu verwenden, wie z. B. `&dyn Trait` oder `Box<dyn Trait>` ( `Rc<dyn Trait>` würde auch funktionieren).

Um mit DSTs zu arbeiten, hat Rust eine bestimmte Eigenschaft namens `Sized` trait, um zu bestimmen, ob die Größe eines Typs zur Kompilierzeit bekannt ist oder nicht. Diese Eigenschaft wird automatisch für alles implementiert, dessen Größe zur Kompilierzeit bekannt ist. Darüber hinaus fügt Rust jeder generischen Funktion implizit eine Grenze für `Sized` hinzu. Das heißt, eine generische Funktionsdefinition wie diese:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

wird eigentlich so behandelt, als hätten wir das geschrieben:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

Standardmäßig funktionieren generische Funktionen nur bei Typen, die zur Kompilierzeit eine bekannte Größe haben. Sie können jedoch die folgende spezielle Syntax verwenden, um diese Einschränkung zu lockern:

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

Ein Merkmal, das an `?Sized` `Sized` ist, bedeutet „ `T` kann Größe haben oder nicht“, und diese Notation setzt die Vorgabe außer Kraft, dass generische Typen zur Kompilierzeit eine bekannte Größe haben müssen. Die `?Trait` mit dieser Bedeutung ist nur für `Sized` verfügbar, nicht für andere Eigenschaften.

Beachten Sie auch, dass wir den Typ des `t` -Parameters von `T` auf `&T` T geändert haben. Da der Typ möglicherweise nicht `Sized` ist, müssen wir ihn hinter einer Art Zeiger verwenden. In diesem Fall haben wir uns für eine Referenz entschieden.

Als nächstes sprechen wir über Funktionen und Schließungen!


[„String Slices“]: ch04-03-slices.html#string-slices
[„Verwenden des Newtype-Musters zum Implementieren externer Merkmale für externe Typen“ gelesen haben.]: ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types