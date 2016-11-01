##SQL Server on Linux

###Fabricio Catae

Acabei de receber acesso aos binários do SQL Server (Preview) para Linux – projeto conhecido como Helsinki. Atualmente estão disponíveis binários para Red Hat, Ubuntu e containers Docker.

<p align = "center">
  <img src="https://msdnshared.blob.core.windows.net/media/2016/08/image352.png">
</p>

A primeira impressão é que tudo funciona normalmente:
* Criei um banco de dados
* Criei uma tabela
* Adicionei constraints
* Inseri dados
* Fiz SELECT
* Consultei as DMV’s

Decidi continuar os testes restaurando backup. Tudo funciona! É realmente incrível saber que há compatibilidade dos arquivos e dos backups.
* Backup Database to Disk (Linux)
* Restore Database from Disk (Windows)

Ainda assim, sempre fica uma dúvida sobre as funcionalidades que estarão presentes no produto. Mas saiba que estão incluídos:
* SQL Audit
* Always On
* Hekaton
* ColumnStore

O produto é realmente incrível. Ele é exatamente o SQL Server 2016! Não há dúvidas de que a Microsoft está confiante sobre a versão Linux. O time do SQLOS está se esforçando e conta com profissionais notórios, como Slava Oks e Bob Dorr, liderando essa iniciativa.

###O futuro é Linux?

Não há dúvidas de que os DBA’s de SQL Server devem aprender Linux.

Entretanto, esse é um produto em Preview e não está pronto. Existem limitações no produto atual e que impedem uma migração Windows para Linux. Certamente as funcionalidades que dependem do Sistema Operacional podem demorar mais tempo para serem implementadas/compatibilizadas. Existem dependências com o LSASS (consequentemente Active Directory) e NTFS. Os serviços adicionais (ex: SQL Agent) não estão disponíveis. Por isso, não vejo o Linux como uma alternativa imediata para quem tem o SQL Server rodando no Windows atualmente.

Por outro lado, a tendência é aumentar os casos de migração Oracle e MySQL para SQL Server, uma vez que o Sistema Operacional deixa de ser um obstáculo.

Reciprocamente, administradores Linux devem aprender SQL Server.

###Quer saber mais?

Se você quiser aplicar, envie o formulário para aprovação.

Formulário: SQL Server on Linux
https://www.microsoft.com/en-us/cloud-platform/sql-server-on-linux

Veja também o relatório do IDC.

SQL Server on Linux: Hath Hell Frozen Over?
http://idcdocserv.com/lcUS41074616

