## Autenticação

As câmeras IP suportam autenticação digest, veja o [RFC 7616](https://datatracker.ietf.org/doc/html/rfc7616) para mais detalhes. 

A autenticação Digest é um esquema de autenticação baseado em desafio-resposta que permite que um cliente se autentique com um servidor web protegido por senha.

Quando um cliente faz uma solicitação HTTP para um recurso protegido por autenticação Digest, o servidor web responde com um código de status HTTP 401, indicando que o acesso não é permitido sem autenticação. Junto com o código de status 401, o servidor também envia um cabeçalho "WWW-Authenticate" contendo informações sobre a proteção utilizada para a senha, bem como um desafio aleatório.
O cliente responde ao desafio enviando um cabeçalho "Authorization" contendo as credenciais de autenticação criptografadas, juntamente com informações sobre o desafio e outras informações necessárias.

**Requisição:** O cliente envia uma solicitação sem informações de autenticação.
```
GET /cgi-bin/magicBox.cgi?action=getLanguageCaps HTTP/1.1
User-Agent: client/1.0
Content-Length: 0
```
**Resposta**: O dispositivo envia uma resposta 401 com parâmetros para o cálculo da autenticação
```
HTTP/1.1 401 Unauthorized
Server: Device/1.0
WWW-Authenticate: Digest realm="Device_00408CA5EA04",
nonce="000562fd20ef95ad", qop="auth",
opaque="5ccc069c403ebaf9f0171e9517f40e41"
```

1 - Quando o cliente não fornece autenticação digest ou quando o cliente calcula a autenticação digest com dados de **"nonce"** expirados, o produto de vídeo responde com um código 401, fornecendo informações para autorização.

2 - Em seguida, o cliente gera **"nc"** e **"cnonce"**, calcula a autenticação digest usando **"username"**, **"password"**, **"método HTTP"**, **"URI"**, **"nc"**, **"cnonce"**, **"realm"** e **"nonce"**, conforme o **RFC 7616**, e envia novamente para o produto de vídeo. Se a autenticação digest estiver correta, o produto de vídeo responderá com o código de status HTTP 200 e os dados da resposta. Se a autenticação digest estiver incorreta, o produto de vídeo responderá com o código de status HTTP 403.

**Requisição:** O cliente envia a solicitação novamente com informações de autenticação (supondo que a senha do usuário admin seja "abcd1234").

```
GET /cgi-bin/magicBox.cgi?action=getLanguageCaps HTTP/1.1
User-Agent: client/1.0
Authorization: Digest username="admin", realm="Device_00408CA5EA04",
nonce="000562fd20ef95ad", nc=00000001, cnonce="0a4f113b", qop="auth",
uri="/cgi-bin/magicBox.cgi?action=getLanguageCaps",
response="dfd0f24bed4c336d20c8f0729dd5dbc8"
opaque="5ccc069c403ebaf9f0171e9517f40e41"
Content-Length: 0
```
**Resposta:** Se as informações de autenticação estiverem corretas, a câmera envia uma resposta `200`.

```
HTTP/1.1 200 OK
Server: Device/1.0
Content-Type: text/plain
Content-Length: <length>
Languages=SimpChinese,English,French
```
**Resposta:** Se as informações de autenticação não estiverem corretas, o dispositivo envia uma resposta `403`.
```
HTTP/1.1 403 Forbidden
Server: Device/1.0
Content-Length: 0 
```
O servidor web verifica se as credenciais estão corretas e, se estiverem, retorna um código de status HTTP 200, indicando que o acesso ao recurso foi permitido.

O motivo pelo qual o servidor web envia um código de status HTTP 401 antes de enviar o código de status HTTP 200 é que o cliente precisa ser informado de que a autenticação é necessária antes de enviar as credenciais de autenticação criptografadas. O código de status HTTP 401 é usado para indicar que o acesso ao recurso é negado sem autenticação.


**Explicação dos parâmetros:**

`username:` Nome de usuário (por exemplo, "admin").\
`realm:` Domínio de autenticação (por exemplo, "example.com").\
`nonce:` Valor fornecido pelo servidor para evitar ataques de repetição.\
`uri:` O recurso solicitado (por exemplo, "/video/resource").\
`response`: Valor calculado que valida a autenticação.\
`qop:` Qualidade de proteção (por exemplo, "auth").\
`nc:` Contador de uso da solicitação (por exemplo, "00000001").\
`cnonce:` Valor gerado pelo cliente para garantir a unicidade.\

O valor do campo response deve ser calculado com base no `username`, `password`, `realm`, `nonce`, `uri`, `nc`, e `cnonce`, seguindo o algoritmo especificado no **RFC 7616**.
