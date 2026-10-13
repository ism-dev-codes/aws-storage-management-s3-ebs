# ☁️ Gerenciamento de Armazenamento na AWS: Amazon S3, Versionamento e EBS Snapshots

Este repositório contém a documentação prática e as evidências do laboratório de gerenciamento, otimização e proteção de armazenamento de objetos e blocos na Amazon Web Services (AWS), utilizando **Amazon S3** e **Amazon EBS**.

---

## 🎯 Objetivos do Laboratório

1. **Amazon S3 (Simple Storage Service):**
   - Criação e configuração de bucket S3.
   - Habilitação de versionamento de objetos para mitigação de perdas acidentais.
   - Sincronização de ficheiros locais com a nuvem utilizando a AWS CLI (`aws s3 sync`).
   - Simulação e controle de exclusões sincronizadas (`--delete`).
   - Recuperação de objetos apagados via *Delete Markers* e IDs de versão (`VersionId`).

2. **Amazon EBS (Elastic Block Store) & EC2:**
   - Associação de perfis de acesso do IAM (*IAM Roles*) a instâncias EC2.
   - Consulta e identificação de volumes EBS via AWS CLI (`describe-instances`).
   - Gerenciamento do ciclo de vida da instância (`stop-instances` / `start-instances`) para garantir a consistência de bloco antes do backup.
   - Criação manual e automação de criação de EBS Snapshots utilizando agendamento via *Cron jobs*.

---

## 🛠️ Passo a Passo de Execução e Evidências

### Parte 1: Gerenciamento de Objetos no Amazon S3

#### 1.1 Criando o Bucket S3
No console do Amazon S3, crie um bucket na região `us-west-2` (Oregon) com namespace globalmente exclusivo.

* **Bucket criado:** `s3-lab183-ism`

![Criar Bucket](Gerenciar%20o%20armazenamento%20%2830%29.png)
![Bucket Criado](Gerenciar%20o%20armazenamento%20%2832%29.png)

#### 1.2 Habilitando o Versionamento de Objetos
Para garantir que edições e exclusões possam ser revertidas, ative o versionamento via AWS CLI:

    aws s3api put-bucket-versioning --bucket s3-lab183-ism --versioning-configuration Status=Enabled

![Versionamento Ativo](Gerenciar%20o%20armazenamento%20%2811%29.png)

#### 1.3 Sincronização Inicial de Ficheiros
Com os ficheiros `file1.txt`, `file2.txt` e `file3.txt` na pasta local `files/`, sincronize a pasta local com o bucket S3:

    # Sincronizar pasta local com o bucket
    aws s3 sync files s3://s3-lab183-ism/files/

    # Listar objetos armazenados no S3
    aws s3 ls s3://s3-lab183-ism/files/

![Upload Inicial CLI](Gerenciar%20o%20armazenamento%20%2814%29.png)
![Arquivos Console S3](Gerenciar%20o%20armazenamento%20%2816%29.png)
![Arquivos CLI S3](Gerenciar%20o%20armazenamento%20%2819%29.png)

#### 1.4 Teste de Exclusão Sincronizada
Remova um ficheiro localmente e propague a exclusão para a nuvem utilizando a flag `--delete`:

    # Apagar ficheiro local
    rm files/file1.txt

    # Sincronizar refletindo a exclusão no S3
    aws s3 sync files s3://s3-lab183-ism/files/ --delete

*Nota:* Como o versionamento está ativo, o S3 aplica um *DeleteMarker* no topo da pilha de versões do objeto em vez de excluir definitivamente os dados.

![Remoção Local](Gerenciar%20o%20armazenamento%20%2820%29.png)
![Sincronização Destrutiva](Gerenciar%20o%20armazenamento%20%2822%29.png)
![Console Pós Exclusão](Gerenciar%20o%20armazenamento%20%2823%29.png)

#### 1.5 Restauração de Objeto Excluído
Consulte o histórico de versões do objeto apagado, recupere o ficheiro via `VersionId` e volte a sincronizá-lo:

    # Listar histórico e identificar o VersionId
    aws s3api list-object-versions --bucket s3-lab183-ism --prefix files/file1.txt

    # Baixar a versão anterior do objeto
    aws s3api get-object --bucket s3-lab183-ism --key files/file1.txt --version-id <VERSION_ID> files/file1.txt

    # Enviar novamente o objeto restaurado para o S3
    aws s3 sync files s3://s3-lab183-ism/files/

![Lista de Versões](Gerenciar%20o%20armazenamento%20%2824%29.png)
![Get Object Version](Gerenciar%20o%20armazenamento%20%2826%29.png)
![Resync Arquivo](Gerenciar%20o%20armazenamento%20%2827%29.png)
![Arquivo Restaurado S3](Gerenciar%20o%20armazenamento%20%2828%29.png)

---

### Parte 2: Proteção de Armazenamento de Blocos com Amazon EBS Snapshots

#### 2.1 Configuração de Permissões IAM na Instância EC2
Associe a role `S3BucketAccess` à instância `Processor` para conceder permissões adequadas de leitura/escrita no S3 e gerenciamento de volumes EC2/EBS:
* **Caminho no console:** EC2 > Instâncias > Selecionar `Processor` > Ações > Segurança > Modificar função do IAM.

![Instâncias EC2](Gerenciar%20o%20armazenamento%20%2834%29.png)
![Modificar Role IAM](Gerenciar%20o%20armazenamento%20%2835%29.png)
![Atribuir IAM Role](Gerenciar%20o%20armazenamento%20%2837%29.png)
![IAM Atualizado](Gerenciar%20o%20armazenamento%20%2836%29.png)

#### 2.2 Desligamento da Instância e Mapeamento de Volume
Conecte-se ao `Command Host` via **EC2 Instance Connect** e execute os comandos para consultar o `VolumeId`, `InstanceId` e interromper a instância com segurança antes da criação do snapshot:

    # Identificar Volume ID do EBS
    aws ec2 describe-instances --filter 'Name=tag:Name,Values=Processor' --query 'Reservations[0].Instances[0].BlockDeviceMappings[0].Ebs.{VolumeId:VolumeId}'

    # Identificar Instance ID
    aws ec2 describe-instances --filters 'Name=tag:Name,Values=Processor' --query 'Reservations[0].Instances[0].InstanceId'

    # Parar a instância e aguardar conclusão
    aws ec2 stop-instances --instance-ids <INSTANCE_ID>
    aws ec2 wait instance-stopped --instance-id <INSTANCE_ID>

![Command Host EC2](Gerenciar%20o%20armazenamento%20%2838%29.png)
![Conectar EC2](Gerenciar%20o%20armazenamento%20%2839%29.png)
![Terminal EC2](Gerenciar%20o%20armazenamento%20%2840%29.png)
![Consulta Volume ID](Gerenciar%20o%20armazenamento%20%2842%29.png)
![Consulta Instance ID](Gerenciar%20o%20armazenamento%20%2843%29.png)
![Stop Instance CLI](Gerenciar%20o%20armazenamento%20%2846%29.png)
![Wait Stopped CLI](Gerenciar%20o%20armazenamento%20%2849%29.png)
![Instância Parada Console](Gerenciar%20o%20armazenamento%20%2850%29.png)

#### 2.3 Geração Manual de EBS Snapshot
Crie o snapshot do volume paralisado e reinicie a máquina após a conclusão:

    # Criar Snapshot
    aws ec2 create-snapshot --volume-id <VOLUME_ID>

    # Aguardar conclusão do Snapshot
    aws ec2 wait snapshot-completed --snapshot-id <SNAPSHOT_ID>

    # Reiniciar a instância
    aws ec2 start-instances --instance-ids <INSTANCE_ID>

![Create Snapshot CLI](Gerenciar%20o%20armazenamento%20%2851%29.png)
![Snapshot Concluído CLI](Gerenciar%20o%20armazenamento%20%2854%29.png)
![Snapshot Console](Gerenciar%20o%20armazenamento%20%2855%29.png)
![Start Instance CLI](Gerenciar%20o%20armazenamento%20%2857%29.png)
![Instância Running Console](Gerenciar%20o%20armazenamento%20%2858%29.png)

#### 2.4 Automação via Cron Job e Limpeza de Tarefas
Verifique o acúmulo de backups automatizados gerados pelo script Python em execução via *Cron*, e ao finalizar o teste, encerre o agendamento:

    # Listar snapshots gerados para o volume
    aws ec2 describe-snapshots --filters "Name=volume-id,Values=<VOLUME_ID>"

    # Desativar a rotina automatizada
    crontab -r

![Describe Snapshots Cron](Gerenciar%20o%20armazenamento%20%2859%29.png)
![Lista Snapshots Cron Console](Gerenciar%20o%20armazenamento%20%2860%29.png)
![Limpeza Crontab](Gerenciar%20o%20armazenamento%20%2862%29.png)

---

## 🏁 Conclusão

Este projeto demonstrou a aplicação prática das melhores diretrizes do **AWS Well-Architected Framework**, unindo resiliência e integridade de dados na camada de armazenamento não estruturado (S3) com rotinas eficientes de backup e disaster recovery na camada de blocos (EBS).
