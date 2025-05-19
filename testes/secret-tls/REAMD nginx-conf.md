O arquivo nginx.conf configura um servidor Nginx que:  
- Escuta conexões HTTP (porta 80) e HTTPS (porta 443).  
- Usa certificados TLS para HTTPS.  
- Responde com uma mensagem simples de texto na rota /.  
  
Para efeitos desse exemplo de uso de Secret TLS pelo nginx é preciso criar esse arquivo (nginx.conf)  
para que seja criado o ConfigMap nginx-config com o comando:  
`kubectl create configmap nginx-config --from-file=nginx.conf`
e antes de aplicar o manifesto nginx-com-secret.yaml