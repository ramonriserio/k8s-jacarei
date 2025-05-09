# JOB
## sleep-job.yaml

Até agora, o tipo de carga de trabalho que estávamos implantando eram, em sua maioria, aplicativos de longa execução, como sites e serviços. Esses aplicativos continuam em execução continuamente e, se algo der errado, eles são reprogramados e começam a rodar novamente.

Por outro lado, há cargas de trabalho que realizam uma tarefa específica e, uma vez concluída essa tarefa, não fazem sentido continuar sendo executadas. Reiniciá-las após sua conclusão seria desnecessário. Um exemplo desse tipo de carga de trabalho seria a realização de backups ou a geração de relatórios diários. Não há necessidade de manter a carga de trabalho de relatórios em execução constantemente — ela só precisa rodar no momento da geração do relatório. Após essa geração, o processo pode desaparecer. Se a tarefa falhar, é possível configurá-la para reiniciar automaticamente ou simplesmente não reiniciar.

O Kubernetes possui um recurso chamado Jobs, que permite a execução dessas cargas de trabalho. Esse recurso pode criar um ou mais pods e rastrear o número de conclusões bem-sucedidas desses pods, garantindo que sejam executados até o final. Poderíamos alcançar um comportamento similar utilizando apenas pods, mas, nesse caso, seria necessário gerenciar manualmente seu ciclo de vida, caso falhem ou sejam reprogramados.

Para ilustrar, vamos executar um job simples que não faz nada além de "dormir" por um minuto. Criamos um arquivo chamado `sleep-job.yaml` e configuramos a política de reinicialização como `Never`. O valor padrão para pods é `Always`, porém, o recurso de jobs do Kubernetes não suporta essa política de reinicialização. Em vez disso, ele suporta apenas dois valores: `Never` e `OnFailure`. Ou seja, os pods nunca serão reiniciados ou só serão reiniciados em caso de falha.

Agora, vamos implantar esse job. Utilizamos o comando:

```
kubectl apply -f sleep-job.yaml
```

Uma vez criado, podemos listar os jobs ou descrevê-los, como faríamos com qualquer outro recurso. Por exemplo:

```
kubectl describe job sleep-job
```

Esse comando fornecerá detalhes sobre o job, como o número de conclusões bem-sucedidas, a configuração de paralelismo (sobre a qual falaremos mais adiante) e os contêineres executados como parte desse trabalho.

Agora, vamos observar o pod criado por esse job. Ele está em execução porque o configuramos para "dormir" por um minuto. Durante 60 segundos, ele ficará em repouso e, depois desse período, a execução será encerrada. No entanto, ele não será automaticamente excluído—o Kubernetes não apagará o pod.

Podemos verificar o status do pod com:

```
kubectl get pods
```

Inicialmente, o status estará marcado como "Em execução" (`Running`), mas, após o tempo configurado, mudará para "Concluído" (`Completed`). Isso acontece de forma semelhante com o próprio recurso de jobs.

Um job permanece ativo até que seja explicitamente excluído. Se deletarmos o job, o pod criado por ele também será removido. Podemos notar que a coluna de conclusões mostra "1 de 1". Por padrão, o Kubernetes define o número de conclusões como "1", mas é possível modificar esse valor.

Por exemplo, digamos que queremos criar um job que execute três ciclos de "sono", em vez de apenas um. Criamos um novo arquivo chamado `three-sleeps.yaml`, semelhante ao anterior, mas definindo o número de conclusões como "3" (dentro de `spec`: `completions: 3`). Agora, implantamos esse job:

```
kubectl apply -f three-sleeps.yaml
```

Podemos observar que o primeiro job ainda está ativo, enquanto o segundo começou e apresenta "0 de 3" conclusões. Assim que o primeiro ciclo terminar, cerca de 63 ou 64 segundos depois, a coluna será atualizada e mostrará "1 de 3". O Kubernetes marca o job como concluído somente após todas as execuções serem bem-sucedidas, sem falhas.

Há também um recurso que permite executar pods em paralelo, utilizando a configuração de paralelismo. No arquivo `three-sleeps-parallel.yaml`, além de definir três conclusões, especificamos `parallelism: 2`. Isso significa que até dois pods podem rodar simultaneamente.

Vamos verificar a execução:

```
kubectl get pods
```

Parece que um dos pods do job `three-sleeps.yaml` foi concluído, enquanto outro ainda está em execução, funcionando sequencialmente. Agora, implantamos o job paralelo:

```
kubectl apply -f three-sleeps-parallel.yaml
```

O Kubernetes criará os pods e permitirá que até dois deles sejam executados ao mesmo tempo, otimizando o processamento dessas cargas de trabalho.