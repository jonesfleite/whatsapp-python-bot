# WhatsApp Bot
Demo Bot para api https://chat-api.com/en/swagger.html

# Oportunidades:
- A saída da lista de comandos
- A saída do ID de bate-papo atual
- A saída do horário real do servidor do bot em execução
- A saída do seu nome
- Envio de arquivos de diferentes formatos (PDF, jpg, doc, mp3, etc.)
- Envio de mensagens de voz pré-gravadas
- Envio de geo-coordenadas (localizações)
- Configurando uma conferência (grupo)

Atenção: Para tornar o bot totalmente funcional, por favor, sempre mantenha seu telefone online. Seu telefone não deve ser usado para o WhatsApp Web ao mesmo tempo.

# Começando
Primeiramente, vamos vincular o WhatsApp ao nosso script de uma vez para que possamos verificar como o código funciona enquanto o escrevemos. Para fazer isso, mudaremos para nossa conta e obteremos um código QR lá. Em seguida, abrimos o WhatsApp no ​​seu celular, vá para Configurações → WhatsApp Web → Digitalize o código QR.

Neste ponto, o URL do WebHook deve ser fornecido para que o servidor acione nosso script para mensagens recebidas. O URL do WebHook é um link para o qual os dados JSON contendo informações sobre as mensagens recebidas ou notificações serão enviados usando o método POST. Portanto, precisamos de um servidor para fazer o bot rodar e esse servidor aceitará e processará essas informações. Ao escrever este artigo, implantamos o servidor usando o microframework FLASK. O servidor FLASK nos permite responder convenientemente às solicitações recebidas e processá-las.
> frasco de instalação pip

Em seguida, clone o repositório para você.
Em seguida, vá para o arquivo ** wabot.py ** e altere as variáveis ​​APIUrl e token para as suas em seu gabinete pessoal
https://app.chat-api.com/instance/

Construtor da classe, o padrão que aceitará JSON, que conterá informações sobre as mensagens de entrada (serão recebidas pelo WebHook e encaminhadas para a classe). Para ver como ficará o JSON recebido, você pode entrar na seção de teste fácil de usar, disponível em seu gabinete. Você também pode testar as solicitações do WebHook nele. https://app.chat-api.com/testing

Você pode visualizar a estrutura JSON na seção de teste do item “Check WebHook”. É necessário executar a verificação e enviar mensagens para o chat do WhatsApp. Você verá o JSON enviado para o WebHook no visor.

Copie este JSON e execute nosso servidor FLASK local com a ajuda de um depurador no editor de código ou através do
> flask run

Para emular a solicitação para o servidor, precisamos enviar uma solicitação POST do JSON, que copiamos na etapa anterior. A solicitação é enviada ao endereço do host local onde o flask está sendo executado. Assim, podemos emular as ações do WebHook e testar a funcionalidade do bot.

# Functions
## send_request 
Usado para enviar solicitações à API do site
```python
 def send_requests(self, method, data):
        url = f"{self.APIUrl}{method}?token={self.token}"
        headers = {'Content-type': 'application/json'}
        answer = requests.post(url, data=json.dumps(data), headers=headers)
        return answer.json()
```
- **method** determina o método da API de bate-papo a ser chamado.
- **data** contém os dados necessários para o envio.


## send_message
Used to send messages to WhatsApp chat
```python
 def send_message(self, chatID, text):
        data = {"chatID" : chatID,
                "body" : text}  
        answer = self.send_requests('sendMessage', data)
        return answer
```
- ChatID – ID of the chat where the message should be sent
- Text – Text of the message

## show_chat_id
Serves to answer the command "chatId". Sends user id to the chat
```python
def show_chat_id(self,chatID):
        return self.send_message(chatID, f"Chat ID : {chatID}")
```
- ChatID – ID of the chat where the message should be sent

## time
Serves to answer the "time" command. Sends the current server time to the chat.
```python
def time(self, chatID):
        t = datetime.datetime.now()
        time = t.strftime('%d:%m:%Y')
        return self.send_message(chatID, time)
```
- ChatID – ID of the chat where the message should be sent

## file 
Serves to answer the command "file". Sends to the chat file, which lies on the server in the specified format
 ```python
def file(self, chatID, format):
        availableFiles = {'doc' : 'document.doc',
                        'gif' : 'gifka.gif',
                        'jpg' : 'jpgfile.jpg',
                        'png' : 'pngfile.png',
                        'pdf' : 'presentation.pdf',
                        'mp4' : 'video.mp4',
                        'mp3' : 'mp3file.mp3'}
        if format in availableFiles.keys():
            data = {
                        'chatId' : chatID,
                        'body': f'https://domain.com/Python/{availableFiles[format]}',                      
                        'filename' : availableFiles[format],
                        'caption' : f'Get your file {availableFiles[format]}'
                    }
            return self.send_requests('sendFile', data)
```
- chatID – ID of the chat where the message should be sent
- format – is the format of the file to be sent. All files to be sent are stored on the server-side.

What's a *data*
- ChatID – ID of the chat where the message should be sent
- Body – direct link to the file to be sent
- Filename – the name of the file
- Caption – the text to be sent out together with the file

Generate request with the **send_requests**  and the **“sendFile”** as a parameter; submit our *data* to it.

## ptt
Used to send a voice message to a chat room.
```python
def ptt(self, chatID):        
            data = {
            "audio" : 'https://domain.com/Python/ptt.ogg',
            "chatId" : chatID }
            return self.send_requests('sendAudio', data)
```
- ChatID – ID of the chat where the message should be sent

What's a *data*
- “audio”  – a direct link to a file in ogg format
-  "chatID" – ID of the chat where the message should be sent

Generate request with the **send_requests**  and the **“sendAudio”** as a parameter; submit our *data* to it.

## geo
Used to send geolocation to a chat room
def geo(self, chatID):
```python
        data = {
                "lat" : '51.51916',
                "lng" : '-0.139214',
                "address" :'Your address',
                "chatId" : chatID
        }
        answer = self.send_requests('sendLocation', data)
        return answer
```
- ChatID – ID of the chat where the message should be sent

What's a *data*
- "сhatID" – ID of the chat where the message should be sent
- “lat” – predefined coordinates
- “lng” – predefined coordinates
- “address” – this is your address or any string you need.

Generate request with the **send_requests**  and the **“sendLocation”** as a parameter; submit our *data* to it.

## group
Used to create a group of bot and user
```python
def group(self, author):
        phone = author.replace('@c.us', '')
        data = {
            "groupName"  :  'Group with the bot Python',
             "phones"  :  phone,
             "messageText"  :  'It is your group. Enjoy'
        }
        answer = self.send_requests('group', data)
        return answer
```
- author –  the JSON body sent by the WebHook with information on the message sender.

What's a *data*
- “groupName” – the conference name when created
- “phones” – all the necessary participants' phone numbers; you can also send an array of several phone numbers
- “messageText” – The first message in the conference

Generate request with the **send_requests**  and the **“group”** as a parameter; submit our *data* to it.

# Incoming message processing
```python
def processing(self):
        if self.dict_messages != []:
            for message in self.dict_messages:
                text = message['body'].split()
                if not message['fromMe']:
                    id  = message['chatId']
                    if text[0].lower() == 'hi':
                        return self.welcome(id)
                    elif text[0].lower() == 'time':
                        return self.time(id)
                    elif text[0].lower() == 'chatid':
                        return self.show_chat_id(id)
                    elif text[0].lower() == 'me':
                        return self.me(id, message['senderName'])
                    elif text[0].lower() == 'file':
                        return self.file(id, text[1])
                    elif text[0].lower() == 'ptt':
                        return self.ptt(id)
                    elif text[0].lower() == 'geo':
                        return self.geo(id)
                    elif text[0].lower() == 'group':
                        return self.group(message['author'])
                    else:
                        return self.welcome(id, True)
                else: return 'NoCommand'
```

Now we need to arrange the bot's logic so it can react to commands and interact with the user as well.
We will call this function for each time we receive data in our WebHook

Do you remember the attribute of our bot dict_messages we created at the beginning? It has dictionaries of messages that we have accepted. The test filters out data not containing any messages. WebHook can receive a request with no message.
```python
 if self.dict_messages != []:
```

We may receive several messages in a single request, and our bot must process them all. To do this, we are going through all the dictionaries which have a dict_messages list.

```python
for message in self.dict_messages:
                text = message['body'].split()
```
We declare a variable text — which will be a list of words contained in our message — after entering the loop. To do this, we turn to the message dictionary using the ['body'] key to get the text of the incoming message and just call the split() function, which allows us to split the text into words.

We then check if the incoming message is not coming from ourselves by referring to the 'fromMe' key which contains True or False values and validates who the message was from. If this test is not performed, the bot may become an infinite recursion.

```python
 if not message['fromMe']:
```

We now get the chat id from the same message dictionary using the ['ChatID'] key. We address the first element of the word list, put it in the lower case so that the bot could react to messages written by UPPERCASE or MiXeD cAsE and then compare it with the commands we need. Next, just parse which command came and call the corresponding functions 
```python
if text[0].lower() == 'hi':
                        return self.welcome(id)
                    elif text[0].lower() == 'time':
                        return self.time(id)
                    elif text[0].lower() == 'chatid':
                        return self.show_chat_id(id)
                    elif text[0].lower() == 'me':
                        return self.me(id, message['senderName'])
                    elif text[0].lower() == 'file':
                        return self.file(id, text[1])
                    elif text[0].lower() == 'ptt':
                        return self.ptt(id)
                    elif text[0].lower() == 'geo':
                        return self.geo(id)
                    elif text[0].lower() == 'group':
                        return self.group(message['author'])
                    else:
                        return self.welcome(id, True)
                else: return 'NoCommand'
```

# Flask 
To process incoming requests to our server we use this function 
```python
@app.route('/', methods=['POST'])
def home():
    if request.method == 'POST':
        bot = WABot(request.json)
        return bot.processing()
```

We'll write the path app.route('/', methods = ['POST']) for it. This decorator means that our home function will be called every time our FLASK server is accessed through a post request by the main path.

We are testing the fact that the server was accessed through the POST method. Now we create an instance of our bot and submit JSON data to it. So now we just call the bot.processing() method that handles requests from our object.
