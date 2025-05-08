# CRONJOB
## cronjob.yaml

Um _CronJob_ cria _Jobs_ em um cronograma repetitivo.

O CronJob serve para executar ações regulares agendadas, como backups, geração de relatórios e assim por diante. Um objeto _CronJob_ é como uma linha de um arquivo _crontab_ (tabela cron) em um sistema Unix. Ele executa um Job periodicamente em um cronograma específico, escrito no formato Cron .

CronJobs têm limitações e peculiaridades. Por exemplo, em certas circunstâncias, um único CronJob pode criar vários Jobs simultâneos.

Quando o plano de controle cria novos Jobs e (indiretamente) Pods para um _CronJob_, o `.metadata.name` do _CronJob_ faz parte da base do nome desses Pods.

Este exemplo de manifesto _CronJob_ (cronjob.yaml) imprime a hora atual e uma mensagem de saudação a cada minuto.

Escrevendo uma especificação CronJob
Sintaxe de programação
O .spec.schedulecampo é obrigatório. O valor desse campo segue a sintaxe do Cron :

 ┌───────────── minute (0 - 59)  
 │┌───────────── hour (0 - 23)  
 ││┌───────────── day of the month (1 - 31)  
 │││┌───────────── month (1 - 12)  
 ││││┌───────────── day of the week (0 - 6) (Sunday to Saturday)  
 │││││                                   OR sun, mon, tue, wed, thu, fri, sat  
 │││││  
 │││││  
 \* * * * *  

Por exemplo, 0 3 * * 1significa que esta tarefa está agendada para ser executada semanalmente em uma segunda-feira às 3h.

Além da sintaxe padrão, algumas macros como @monthly:

| Entrada               | Descrição                                             | Equivalente a    |
|----------------------|-----------------------------------------------------|----------------|
| @yearly (ou @anualmente) | Executa uma vez por ano à meia-noite de 1º de janeiro | `0 0 1 1 *`    |
| @mensal             | Executa uma vez por mês à meia-noite do primeiro dia do mês | `0 0 1 * *`    |
| @semanalmente       | Executa uma vez por semana à meia-noite de domingo    | `0 0 * * 0`    |
| @daily (ou @midnight) | Executa uma vez por dia à meia-noite                 | `0 0 * * *`    |
| @de hora em hora    | Executa uma vez por hora no início da hora            | `0 * * * *`    |

Para gerar expressões de agendamento do CronJob, você também pode usar ferramentas da web como [crontab.guru](https://crontab.guru).

**Fonte**: [https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)