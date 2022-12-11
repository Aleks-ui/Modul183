# Lern-Bericht
Aleksandar Veselinov

## Einleitung

Die Aufgabenstellung zur SL-InterpreterInjection wurde in zwei Teile aufgeteilt, wobei das zweite Teil mehr im Fokus gesetzt wird. 
In dieser Aufgabe geht es darum, die Manipulation von eingegebene Daten bei der Insecure-App zu verhindern, die als Anweisungen gelesen werden, wenn man sie ausführen möchte.
In diesen Teil wurde es spezifisch angegebn die XML und SQL-Injection zu verwenden. In diesem Lernbericht werde ich auf die Reparaturen davon eingehen.

## Was habe ich gelernt?

Ich lernte bei diesen Auftrag, wie die Interpreter funktionieren und wie man Daten fälschlicherweise interpretieren kann, um sie zu manipulieren.

## Beschreibung

In diesem Beispiel werde ich Ihnen das Problem kurz beschreiben und wie man sie entsprechend korrigiert.
Die Interpreter von SQL und XML anhand ihre Tags (XML) und Streams (SQL). 
Würde man Daten von einer Webseite in einer Text-Eingabe abschicken, wo sie danach in einem XML-Dokument gespeichert werden, so kann man bereits die XML-Datei manipulieren.

### XML
#### Beispiel Admin
```XML
<user id="2">
        <username>user</username>
        <password>uS3rP8ss</password>
        <isadmin>0</isadmin>
    </user>
```
#### nach der Eingabe
```XML
    <user id="2">
        <username>user</username>
        <password>
🔴         hacked
🔴       </password>
🔴       <isAdmin>1</isAdmin>
🔴   </user>
🔴   <user id="100">
🔴        <username>hacker123</username>
🔴        <password>whoKnow$</password>
        <isadmin>0</isadmin>
    </user>
```

Hier gibt der User im Eingabe-Feld das was mit roten Punkten markiert ist ein und wird danach beim XML-File aufgenommen und eingelesen.

Um dies zu verhindern, muss man die Eingabe bzw. Ausgabe 'escapen'. Es wird folgendes getan:
```XHMTL
<h:outputText value="#{Login.detail}" escape="true"/>
```
Escape einfügen und auf true setzen.

### SQL
Bei der InsecureApp kann man neue News erstellen, die bei der Datenbank gespeichert werden. 

![image](https://user-images.githubusercontent.com/85628102/206927290-803ee599-06dd-4d02-8e0b-dda851488dec.png)

Gibt man bei den Details die Injection ein, so kann man die Datenbank leicht manipulieren. So werden andere Nutzer etwas unerwünschtes oder falsches sehen.
Das Ganze läuft im Java-Code ab.

#### Beispiel News vor dem Injection-Schutz
```Java
public int insert(News news) {
        final String sql = "INSERT INTO news (posted, header, detail, author, is_admin_news) VALUES ('" + new java.sql.Timestamp(news.getPosted().getTime()) + "','" + news.getHeader() + "','" + news.getDetail() + "','" + news.getAuthor() + "'," + (news.getIsAdminNews() ? "1" : "0") + ")";
        int id = 0;

        try (Statement stmt = DbAccess.getConnection().createStatement()) {
            stmt.execute(sql);
            id = stmt.getGeneratedKeys().getInt(1);
        } catch (SQLException e) {
            Logger.getLogger(NewsDAO.class.getName()).log(Level.SEVERE, null, e);
        }

        return id;

    }
```
Damit man die Manipulation der Datenbank-Daten verhindert wird,
wird das Isert-Statement mit unbekannten Values gesetzt und mit dem Prepared-Statement auf die gefährliche Injection vorbereitet.

```Java
    public int insert(News news) {
🟢      final String sql = "INSERT INTO news (posted, header, detail, author, is_admin_news) VALUES (?,?,?,?,?)";
        int id = 0;

        Connection conn = DbAccess.getConnection(); 
🟢      try ( PreparedStatement pstmt = conn.prepareStatement(sql)) {
            id = pstmt.getGeneratedKeys().getInt(1);
🟢            pstmt.setDate(1, new java.sql.Date(news.getPosted().getTime()));
🟢            pstmt.setString(2, news.getHeader());
🟢            pstmt.setString(3, news.getDetail());
🟢            pstmt.setString(4, news.getAuthor());
🟢            pstmt.setBoolean(5, news.getIsAdminNews());
🟢            pstmt.execute();
        } catch (SQLException e) {
            Logger.getLogger(NewsDAO.class.getName()).log(Level.SEVERE, null, e);
        }

        return id;

    }
```
Bei dem String 'sql' wird die Daten als Unbekannte gesetzt. Erst später benutzt man den PreparedStatement. 
Dieser sagt einfach, dass bei diesen Datensätzen aufgepasst werden soll was entgegengenommen wird und der Interpreter unterscheidet was ein Datensatz ist und was eine Anweisung ist. Hätte es dies nicht gegeben, hätte ein Hacker eine News erstellen können, die gekennzeichnet wurde, dass es von einem Admin veröffentlicht wurde.


## Verifikation

Ich habe gelernt, wie man Speicherverwaltungen und dessen Interpreters vor Injections schützt.

# Reflektion zum Arbeitsprozess

Ich konnte die dazugehörigen Lektionen bei Hacksplaining noch einmal durchnehmen als Repetition. Dadurch konnte ich mein Wissen wieder auffrischen und es zugleich wieder testen.
Ich habe die Übungen erneut gemacht und kam sehr gut voran. Dies hat mir gezeigt, dass ich dieses Thema gut verstanden habe und mich einigermassen auf die Leistungsbeurteilung vorbereitet habe.

Ich habe im Verlauf des Repetierens der Injection nicht allzu gut verstanden, wie XML-Injections verhindert werden.
Im Arbeitsblatt wird nur erwähnt, dass man den Escaping ausschalten muss, aber ansonsten ist es nicht so umfangreich wie bei der SQL-Injection-Verhinderung.

**VBV**: Beim nächsten Mal eventuell dem Lehrer nachfragen vor der Leisungsbeurteilung, wie etwas funktioniert, falls es welche Unklarheiten gibt.
