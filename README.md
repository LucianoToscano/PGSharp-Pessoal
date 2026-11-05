# PGSharp-Pessoal — COMPILADOR PÚBLICO

Este repositório é **somente a central de compilação**, não o lugar onde desenvolvemos nem guardamos dados pessoais.

| Repositório | Função |
| --- | --- |
| [PGSharp-Privado](https://github.com/LucianoToscano/PGSharp-Privado) | Código-fonte, editor de favoritos, patch, testes; privado |
| [PGSharp-Pessoal](https://github.com/LucianoToscano/PGSharp-Pessoal) | GitHub Actions manual para compilar APK experimental; público |

## Workflow

[Executar gerador](https://github.com/LucianoToscano/PGSharp-Pessoal/actions/workflows/gerar-apk.yml)

- **validar:** lê a fonte privada e executa testes; não gera APK.
- **apk:** usa fonte privada + APK base de uma release privada, recompila e assina com sua própria chave.
- O APK gerado é colocado em **artefato 7z criptografado**, com retenção de 1 dia.
- Nenhum APK original, DAT, JSON de inventário ou segredo deve ser enviado ao repositório público.

## Configuração obrigatória (uma vez)

Em **Settings > Secrets and variables > Actions > New repository secret**, neste repositório público, crie:

1. PRIVATE_REPO_READ_TOKEN — token GitHub **fine-grained**, escopo somente PGSharp-Privado, permissão Contents: Read. O workflow precisa ler o código e obter o APK-base por uma release privada.
2. PGSHARP_SIGNING_KEYSTORE_B64 — representação Base64 de **uma chave de assinatura criada por você**; a chave precisa ser mantida para todas as futuras atualizações.
3. PGSHARP_SIGNING_PASSWORD — senha da chave.
4. PGSHARP_SIGNING_ALIAS — alias, por exemplo pgspessoal.
5. PGSHARP_ARTIFACT_PASSWORD — senha longa de criptografia do arquivo 7z.

### Como criar sua chave localmente
Com JDK instalado:

~~~bash
keytool -genkeypair -v -keystore pgspessoal.jks -storetype PKCS12 -alias pgspessoal -keyalg RSA -keysize 3072 -validity 10000 -dname "CN=PGS Pessoal,O=Uso Particular,C=BR"
~~~

Copie o arquivo .jks para local seguro, sem enviá-lo a git. No PowerShell, converta-o para o segredo Base64:

~~~powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\caminho\pgspessoal.jks")) | Set-Clipboard
~~~

Cole a string no secret PGSHARP_SIGNING_KEYSTORE_B64. Para gerar atualizações instaláveis sobre **o próprio APK personalizado**, use sempre a **mesma chave**, e não uma chave nova a cada build. O APK modificado não poderá atualizar por cima a instalação original com assinatura diferente.

### APK base privado
Crie a release **base-apk** em [PGSharp-Privado/releases/new](https://github.com/LucianoToscano/PGSharp-Privado/releases/new), anexando **sua cópia legítima** do APK como asset chamado **original.apk**. Ela ficará acessível somente aos colaboradores do repositório privado; não envie o arquivo para o repositório público.

### Gerar
Abra **Actions > PGSharp Pessoal - validação e APK experimental > Run workflow**:
1. modo validar — checa se o acesso à fonte e os testes estão corretos.
2. modo apk — monta, assina e empacota o APK experimental.

Baixe o artefato criptografado da execução e extraia com a senha PGSHARP_ARTIFACT_PASSWORD. Cuidado: essa senha não é recuperável pelo GitHub.

## Situação atual
A fonte experimental está no repositório privado; workflow de build disponível. **Ainda não há APK compilado, instalado ou testado neste fluxo.** Só execute após configurar os segredos e o APK base. Modificações do aplicativo de terceiros podem deixar login, atualização, integração Unity e verificações de integridade inoperantes e trazer riscos para a conta de jogo.
