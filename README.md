# ☁️ Gerenciamento de Armazenamento na AWS: Amazon S3, Versionamento e EBS Snapshots

> ℹ️ **NOTE:** Este é o repositório desenvolvido por **Ismael Santos de Medeiros**.

Projeto prático com o objetivo de demonstrar o gerenciamento, otimização e proteção de armazenamento de objetos e blocos na Amazon Web Services (AWS), utilizando o **Amazon S3** e o **Amazon EBS**. O projeto abrange a criação de buckets S3, habilitação de versionamento, sincronização local-nuvem via AWS CLI (`aws s3 sync`), recuperação de objetos excluídos via *Delete Markers*, associação de *IAM Roles* a instâncias EC2, interrupção controlada de instâncias para consistência e a geração manual e automatizada (via *Cron jobs*) de **EBS Snapshots**.

---

## 💻 Tecnologias utilizadas no projeto

- [Amazon S3 (Simple Storage Service)](https://aws.amazon.com/s3/) — Armazenamento de objetos de alta durabilidade e escala
- [Amazon EBS (Elastic Block Store)](https://aws.amazon.com/ebs/) — Armazenamento em bloco persistente para instâncias EC2
- [Amazon EC2](https://aws.amazon.com/ec2/) — Instâncias virtuais para execução de aplicações e tarefas administrativas
- [AWS CLI](https://aws.amazon.com/cli/) — Interface de linha de comando para automação e gerenciamento de recursos AWS
- [AWS IAM (Identity and Access Management)](https://aws.amazon.com/iam/) — Gerenciamento de papéis (*Roles*) e permissões de acesso
- [Linux Cron Jobs & Python](https://www.linux.org/) — Agendamento e automação de backups periódicos de snapshots

---

## ✨ Como foi feito ?

- Criado e configurado um bucket Amazon S3 (`s3-lab183-ism`) na região `us-west-2` (Oregon).
- Habilitado o versionamento de objetos no S3 via AWS CLI (`aws s3api put-bucket-versioning`) para prevenção de perdas acidentais de dados.
- Sincronizados arquivos locais com o bucket S3 utilizando o comando `aws s3 sync`.
- Simulado o cenário de exclusão sincronizada utilizando a flag `--delete`, demonstrando o uso de *Delete Markers*.
- Restaurado o arquivo apagado recuperando uma versão anterior via `VersionId` através da AWS CLI.
- Associada a *IAM Role* `S3BucketAccess` à instância EC2 `Processor` para concessão de permissões de acesso e gerenciamento.
- Mapeados o `VolumeId` e `InstanceId` da instância via CLI e parada a execução da máquina (`stop-instances`) para garantir consistência dos dados em bloco.
- Criado manualmente um **EBS Snapshot** do volume paralisado com validação do estado `completed`.
- Reiniciada a instância EC2 e validada a automação de backups agendados via script Python e *Cron job*.
- Limpa a rotina automatizada do *Cron* ao término das validações de segurança e auditoria.

---

## 📚 Materiais

- Documentação do Amazon S3: [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- Guia de Versionamento S3: [Using versioning in S3 buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html)
- Guia do AWS CLI para S3: [AWS CLI S3 Commands](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/index.html)
- Documentação de EBS Snapshots: [Amazon EBS Snapshots Guide](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-snapshots.html)

---

## 🛠️ Instruções de execução

Para replicar os procedimentos de gerenciamento de armazenamento S3 e backup EBS, siga as etapas detalhadas abaixo.

- 🤖 1. No console do Amazon S3, inicie o processo de criação de um novo bucket na região `us-west-2` (Oregon).

![Criar Bucket](Gerenciar%20o%20armazenamento%20%2830%29.png)

- 🤖 2. Valide a criação do bucket `s3-lab183-ism` no painel principal do Amazon S3.

![Bucket Criado](Gerenciar%20o%20armazenamento%20%2832%29.png)

- 🤖 3. No terminal, habilite o versionamento do bucket S3 utilizando o comando `aws s3api put-bucket-versioning`.

![Versionamento Ativo](Gerenciar%20o%20armazenamento%20%2811%29.png)

- 🤖 4. Execute a sincronização dos arquivos locais com a nuvem utilizando o comando `aws s3 sync files s3://s3-lab183-ism/files/`.

![Upload Inicial CLI](Gerenciar%20o%20armazenamento%20%2814%29.png)

- 🤖 5. Acesse o console do S3 e confirme a presença dos arquivos sincronizados no bucket.

![Arquivos Console S3](Gerenciar%20o%20armazenamento%20%2816%29.png)

- 🤖 6. No terminal, liste os objetos no S3 com `aws s3 ls` para confirmar o upload completo.

![Arquivos CLI S3](Gerenciar%20o%20armazenamento%20%2819%29.png)

- 🤖 7. Remova um arquivo localmente (`rm files/file1.txt`) para preparar o teste de exclusão sincronizada.

![Remoção Local](Gerenciar%20o%20armazenamento%20%2820%29.png)

- 🤖 8. Propague a exclusão para o S3 executando o comando `aws s3 sync` com a flag `--delete`.

![Sincronização Destrutiva](Gerenciar%20o%20armazenamento%20%2822%29.png)

- 🤖 9. Verifique no console do S3 que o arquivo foi ocultado da exibição padrão.

![Console Pós Exclusão](Gerenciar%20o%20armazenamento%20%2823%29.png)

- 🤖 10. Consulte o histórico de versões do arquivo apagado via `aws s3api list-object-versions` para identificar o `VersionId`.

![Lista de Versões](Gerenciar%20o%20armazenamento%20%2824%29.png)

- 🤖 11. Baixe a versão anterior do arquivo utilizando `aws s3api get-object` passando o `VersionId`.

![Get Object Version](Gerenciar%20o%20armazenamento%20%2826%29.png)

- 🤖 12. Execute novamente a sincronização local-nuvem com `aws s3 sync` para restaurar o arquivo no S3.

![Resync Arquivo](Gerenciar%20o%20armazenamento%20%2827%29.png)

- 🤖 13. Confirme no console do S3 a recuperação bem-sucedida do arquivo `file1.txt`.

![Arquivo Restaurado S3](Gerenciar%20o%20armazenamento%20%2828%29.png)

- 🤖 14. Acesse o painel de instâncias do Amazon EC2 e selecione a instância `Processor`.

![Instâncias EC2](Gerenciar%20o%20armazenamento%20%2834%29.png)

- 🤖 15. Acesse o menu de ações de segurança para modificar a função do IAM da instância.

![Modificar Role IAM](Gerenciar%20o%20armazenamento%20%2835%29.png)

- 🤖 16. Confirme a atribuição da IAM Role `S3BucketAccess` à instância
