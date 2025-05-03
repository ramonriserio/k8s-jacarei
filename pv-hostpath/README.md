# Configurando um Pod para usar um Volume Persistente para armazenamento

Aqui está o resumo do processo:

1. Você, como administrador do cluster, faz a criação de um PersistentVolume suportado por armazenamento físico. Você não associa o volume a nenhum Pod.  
2. Você, agora assumindo o papel de desenvolvedor/usuário do cluster, faz a criação de um PersistentVolumeClaim que é automaticamente vinculado ao Volume Persistente adequado.  
3. Você cria um Pod que usa o PersistentVolumeClaim acima para armazenamento.

## Antes de você começar
- Você precisa ter um cluster Kubernetes e a ferramenta de linha de comando kubectl configurada para se comunicar com seu cluster.

## 1. Crie um Volume Persistente
Neste tutorial, você cria um Volume Persistente do tipo `hostPath`. O Kubernetes suporta `hostPath` para desenvolvimento e teste em um cluster. Um Volume Persistente do tipo 'hostPath' usa um arquivo ou diretório no nó do Pod para emular um armazenamento.

Em um cluster de produção, você não usaria hostPath. Em vez disso um administrador de cluster provisionaria um recurso de rede, como um disco persistente do Google Compute Engine, um NFS compartilhado, ou um volume do Amazon Elastic Block Store. Administradores podem também usar classes de armazenamento para incializar provisionamento dinâmico.

Aqui está o arquivo de configuração para o Volume Persistente hostPath:

```
# pv-volume.yaml
# Criando um volume persistente (PersistentVolume)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  persistentVolumeReclaimPolicy: Retain   # Também pode ser Recycle ou Delete
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

O arquivo de configuração especifica que o volume está no diretório `/mnt/data` do nó do cluster. A configuração também especifica um tamanho de 1 gibibytes e um modo de acesso `ReadWriteOnce`, o que significa que o volume pode ser montado como leitura-escrita por um único nó. Define o nome da StorageClass como `manual` para o PersistentVolume, que será usado para vincular requisições PersistentVolumeClaim à esse Volume Persistente.

Crie o Volume Persistente:
```
kubectl apply -f https://k8s.io/examples/pods/storage/pv-volume.yaml
```

Veja informações do Volume Persistente:
```
kubectl get pv pv-volume
```

A saída mostra que o PersistentVolume tem um `STATUS` de `Available`. Isto significa que ainda não foi vinculado a um PersistentVolumeClaim.

```
NAME             CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
pv-volume        10Gi       RWO           Retain          Available             manual                   4s
```

## 2. Crie um PersistentVolumeClaim

O próximo passo é criar um `PersistentVolumeClaim`. Pods usam `PersistentVolumeClaims` para requisitar armazenamento físico. Neste exercício, você vai criar um PersistentVolumeClaim que requisita um volume com pelo menos três gibibytes, com acesso de leitura-escrita para pelo menos um nó.

Aqui está o arquivo de configuração para o `PersistentVolumeClaim`:

```
# pv-claim.yaml
# Criando uma PersistentVolumeClaim (Requisição de Volume Persistente)
# Antes de executar esse manifesto é preciso criar um PersistentVolume com o manifesto pv-volume.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Crie o `PersistentVolumeClaim`:
```
kubectl apply -f https://k8s.io/examples/pods/storage/pv-claim.yaml
```

Após criar o `PersistentVolumeClaim`, o `control plane` do Kubernetes procura por um Volume Persistente que satisfaça os requerimentos reivindicados. Se o `control plane` encontrar um Volume Persistente adequado, com a mesma classe de armazenamento, ele liga o volume requisitado.

Olhe novamente o PersistentVolume:
```
kubectl get pv pv-volume
```

Agora a saída mostra um `STATUS` de `Bound`.

```
NAME             CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM              STORAGECLASS   REASON    AGE
pv-volume        10Gi       RWO           Retain          Bound     default/pv-claim   manual                   2m
```

Olhe para o `PersistentVolumeClaim`:

```
kubectl get pvc pv-claim
```

A saída mostra que o `PersistentVolumeClaim` está vinculado ao seu PersistentVolume, `pv-volume`.

```
NAME            STATUS    VOLUME      CAPACITY   ACCESSMODES   STORAGECLASS   AGE
pv-claim        Bound     pv-volume   10Gi       RWO           manual         30s
```

## 3. Crie um Pod
O próximo passo é criar um Pod que usa o seu `PersistentVolumeClaim` como um volume.

Aqui está o arquivo de configuração para o Pod:

```
# pv-pod.yaml
# Criando um Pod que usa um PersistentVolumeClaim como um volume.
# Antes de executar esse manifesto é preciso reivindicar  um PersistentVolume com o manifesto pv-claim.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pv-claim
  containers:
  - name: pv-container
    image: nginx
    ports:
      - containerPort: 80
        name: http-server
    volumeMounts:
      - mountPath: /usr/share/nginx/html
        name: pv-storage
```

Note que o arquivo de configuração do Pod especifica um `PersistentVolumeClaim`, mas não especifica um `PersistentVlume`. Do ponto de vista do Pod, a reivindicação é de um volume.

Crie o Pod:
```
kubectl apply -f https://k8s.io/examples/pods/storage/pv-pod.yaml
```

Verifique se o container no Pod está executando:
```
kubectl get pod pv-pod
```

Acesse o nó onde o pod pv-pod foi criado e verifique a existência do diretório `/mnt/data`.
Depois crie o arquivo index.html da seguinte forma:
```
echo "Hello Jacarei" > /mnt/data/index.html
```

Volte para o nó master e abra o shell do container executando no seu Pod, com o seguinte comando:
```
kubectl exec -it pv-pod -- /bin/bash
```

No seu shell, verifique se o nginx está servindo o arquivo `index.html` do volume do `hostPath`:
```
# Certifique-se de executar esses 3 comandos dentro do shell, na raiz que vem da
# execução "kubectl exec" do passo anterior
apt update
apt install curl
curl http://localhost/
```

A saída mostra o texto que você escreveu no arquivo `index.html` no volume do `hostPath`:
```
Hello Jacarei
```

Se você vir essa mensagem, configurou com sucesso um pod para usar o armazenamento de um `PersistentVolumeClaim.`


**Fonte:** https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/