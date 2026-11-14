# MAPA — Motor Analysis & Performance Academy

O MAPA é uma plataforma educacional para estudar calibração de motores. Alunos e professores podem selecionar um motor, editar mapas de combustível e ignição e comparar os efeitos no dinamômetro virtual. Professores organizam turmas, propõem desafios e acompanham o desempenho dos alunos.

O simulador usa um modelo didático para relacionar os ajustes aos resultados de uma puxada. As curvas ajudam a explorar conceitos de calibração e devem ser interpretadas como estimativas do modelo.

## Tecnologias utilizadas

| Camada | Tecnologias |
| --- | --- |
| Frontend | Angular 21, TypeScript, RxJS, PrimeNG e Chart.js |
| Backend | Java 21, Spring Boot 4, Spring MVC, Spring Security com JWT, Spring Data JPA e Gradle |
| Dados | PostgreSQL, com Supabase na implantação |
| Integrações | Resend para recuperação de senha por e-mail e Springdoc/OpenAPI para documentação da API |
| Qualidade | JUnit, Mockito e H2 no backend, Vitest no frontend e GitHub Actions para as verificações do backend |
| Implantação | Vercel para o frontend e Render para a API |

## Como clonar

O repositório principal reúne três submódulos. Clone com a opção `--recurse-submodules` para baixar o código e a documentação.

```bash
git clone --recurse-submodules https://github.com/ricardosb25/MAPA.git
cd MAPA
```

Se já clonou o repositório principal, inicialize os submódulos com `git submodule update --init --recursive`.

| Diretório | Conteúdo |
| --- | --- |
| [MAPA-Frontend](MAPA-Frontend) | Aplicação web |
| [MAPA-Backend](MAPA-Backend) | API, persistência e modelo do dinamômetro |
| [MAPA-Docs](MAPA-Docs) | Documentação técnica e scripts SQL |

## Como executar localmente

Você precisa de Java 21, Node.js 22.12 ou superior na linha 22.x, npm e um banco PostgreSQL acessível. Crie o banco antes de iniciar a API. O Hibernate cria ou atualiza as tabelas na inicialização, conforme a configuração atual.

### Backend

Use [o arquivo de exemplo](MAPA-Backend/.env_sample) como referência. Defina as variáveis abaixo no terminal ou na configuração de execução da IDE. Os comandos de inicialização usam as variáveis do processo. Copiar o exemplo para `.env` por si só não carrega os valores no Spring.

| Variável | Uso |
| --- | --- |
| `DB_HOST`, `DB_PORT`, `DB_NAME` | Host, porta e nome do banco PostgreSQL. Os padrões são `localhost`, `5432` e `postgres`. |
| `DB_USER`, `DB_PASSWORD` | Credenciais de conexão ao banco. |
| `JWT_SECRET` | Segredo usado para assinar os tokens de sessão. Use um valor aleatório com pelo menos 32 bytes em UTF-8. |
| `CORS_ALLOWED_ORIGINS` | Origens autorizadas a acessar a API. Para o Angular local, defina `http://localhost:4200`. |
| `PORT` | Porta da API. O padrão é `8080`. |
| `JWT_EXPIRATION_MS` | Duração do token em milissegundos. O padrão é `86400000` (24 horas). |
| `ADMIN_EMAIL`, `ADMIN_PASSWORD`, `ADMIN_NAME` | E-mail e senha para criar o administrador inicial, caso a conta ainda não exista. O nome é opcional. |
| `RESEND_API_KEY`, `RESEND_FROM_EMAIL` | Chave e remetente autorizado para habilitar o envio de e-mails de recuperação pelo Resend. |
| `FRONTEND_BASE_URL` | Endereço da interface usado nos links de recuperação de senha. O padrão é `http://localhost:4200`. |

No PowerShell, entre em `MAPA-Backend`, configure a conexão e inicie a API. Substitua os valores de exemplo pelas credenciais do seu banco.

```powershell
cd MAPA-Backend
$env:DB_HOST = 'localhost'
$env:DB_PORT = '5432'
$env:DB_NAME = 'mapa'
$env:DB_USER = 'postgres'
$env:DB_PASSWORD = '<senha do PostgreSQL>'
$env:JWT_SECRET = [Guid]::NewGuid().ToString('N') + [Guid]::NewGuid().ToString('N')
$env:CORS_ALLOWED_ORIGINS = 'http://localhost:4200'
.\gradlew.bat bootRun
```

Em Linux ou macOS, exporte as mesmas variáveis e execute `./gradlew bootRun`. A API local fica em `http://localhost:8080/api/v1` e o Swagger em `http://localhost:8080/swagger-ui.html`.

### Frontend

Em outro terminal, a partir da raiz do repositório:

```bash
cd MAPA-Frontend
npm ci
npm start
```

Abra `http://localhost:4200`. O ambiente de desenvolvimento aponta para a API local. Se mudar a porta da API, ajuste `apiUrl` no [ambiente de desenvolvimento](MAPA-Frontend/src/environments/environment.development.ts). O build de produção usa o [ambiente de produção](MAPA-Frontend/src/environments/environment.prod.ts).

## Principais funcionalidades

1. **Conta e acesso.** Alunos e professores se cadastram com aceite versionado dos termos. O login inicia uma sessão autenticada por JWT. Em Configurações, esses perfis podem alterar e-mail e senha e consultar os termos de privacidade.
2. **Garagem.** Selecione um motor do catálogo para usar na bancada virtual. Cilindrada, taxa de compressão, tipo de aspiração e corte de giro entram no modelo da simulação.
3. **Calibração.** Escolha a carga na linha e a rotação na coluna. A célula representa avanço em graus no mapa de ignição ou VE (eficiência volumétrica) no mapa de combustível. Os ajustes são salvos por conta e motor. A grade termina no corte cadastrado, e o usuário pode restaurar os valores padrão do mapa.
4. **Dinamômetro virtual.** A puxada usa os mapas efetivos em plena carga e calcula curvas de torque, potência e AFR (razão ar–combustível). Uma puxada íntegra chega ao corte exato. Mistura pobre, detonação ou excesso de giro podem interrompê-la. O histórico guarda o resumo da simulação.
5. **Turmas e desafios.** O professor cria uma turma e compartilha seu código de matrícula. O aluno seleciona um desafio e simula com o motor exigido. Para concluir, a mesma puxada deve atingir a meta de potência e terminar no estado de motor exigido.
6. **Desempenho do professor.** O painel agrega matrículas, tentativas e conclusões das turmas sob responsabilidade do professor. A taxa de conclusão compara as conclusões às oportunidades de conclusão, calculadas a partir das matrículas e dos desafios de cada turma.
7. **Administração.** O administrador gerencia contas, consulta os logs de auditoria e pode restaurar as bases padrão. A restauração de motores remove os desafios e demais registros vinculados aos motores anteriores. A restauração de mapas substitui os valores de fábrica e remove ajustes individuais.
8. **Aulas.** A área reúne explicações e exercícios sobre ciclo Otto, carga, RPM, avanço, VE, calibração e matemática do simulador.

## Testes e documentação

| Onde | Comando | O que verifica |
| --- | --- | --- |
| Backend (Windows) | `.\gradlew.bat test` | Testes unitários |
| Backend (Windows) | `.\gradlew.bat integrationTest` | Fluxos com Spring, HTTP e banco H2 |
| Frontend | `npm test -- --watch=false` | Testes Angular com Vitest |

Em Linux e macOS, use `./gradlew` para as tarefas do backend. O [workflow de CI](MAPA-Backend/.github/workflows/ci.yml) executa as suítes do backend em jobs separados e inclui uma etapa de Checkstyle.

Consulte a [documentação técnica](MAPA-Docs/Texts/Detailed-Doc.md) para detalhes sobre arquitetura, endpoints, persistência e cálculos. O [script de demonstração do dashboard](MAPA-Docs/SQL/teacher-dashboard-demo.sql) traz dados fictícios para popular um ambiente de demonstração.
