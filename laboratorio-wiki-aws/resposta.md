# 📝 Resposta do Laboratório: A Wiki Perdida dos Arquivos Corporativos

> Preencha este arquivo com a sua proposta de solução.
>
> Sua resposta deve explicar como transformar os documentos brutos da pasta `raw/` em uma Wiki Corporativa Inteligente, pesquisável e segura usando apenas serviços da AWS.

---

## 👤 Identificação

**Nome:**  
Desafio DIO - Wiki Corporativa Inteligente

**Data:**  
12/08/2026

**Link do repositório:**  
https://github.com/wsawebmaster/dio-desafio-github/tree/main/laboratorio-wiki-aws

---

# ✅ Quest 1: O Mapa dos Arquivos Perdidos

## 1.1 Formatos encontrados na pasta `raw/`

Descreva quais tipos de arquivos existem dentro da pasta `raw/`.

```md
Exemplo de como responder, com o formato e o que ele implica:
- <extensao>: <nasce digital ou precisa de OCR?>, <o que da para extrair>
```

> Abra a pasta e liste o que voce encontrou de fato. Esta quest avalia a sua
> leitura do acervo, entao a resposta certa e a que corresponde aos arquivos.

**Sua resposta:**

```md
- ata_reuniao_vendas_sa.pdf: PDF digital com camada de texto, extração de decisões, participantes, datas e próximos passos
- ata_resultados_vendas_novos_dados.png: Imagem escaneada de documento, requer OCR para extração de informações tabulares e texto
- vendas_sa_dados_ficticios_laboratorio.csv: Arquivo de dados estruturado, contém 240 registros de oportunidades de vendas com 19 colunas, já estruturado e pronto para análise
```

---

## 1.2 Principais desafios encontrados

Explique quais dificuldades esses documentos podem apresentar.

```md
Exemplo:
- Arquivos sem padrão de nomenclatura
- Documentos escaneados com baixa qualidade
- Textos manuscritos ou parcialmente ilegíveis
- Atas com estruturas diferentes
- Informações importantes espalhadas em vários formatos
```

**Sua resposta:**

```md
- Documentos em formatos heterogêneos (PDF, imagem, CSV) misturados na mesma pasta sem padrão de nomenclatura
- Imagem escaneada pode ter baixa qualidade de resolução, afetando precisão do OCR
- PDFs e imagens contêm informações dessestruturadas (texto corrido), enquanto CSV é estruturado
- Atas podem ter estruturas diferentes de layout e organização de conteúdo
- Metadados como datas, responsáveis e decisões estão implícitos nos documentos, não em campos estruturados
- Dados estão espalhados em múltiplos arquivos sem consolidação ou linking
- Necessidade de OCR para alguns documentos pode introduzir erros de reconhecimento
- Informações sensíveis (participantes, decisões confidenciais) podem estar misturadas sem classificação
```

---

## 1.3 Informações importantes a serem extraídas

Liste quais informações precisam ser identificadas para transformar os documentos em conhecimento pesquisável.

**Sua resposta:**

```md
- **Entidades de Negócio**: Nomes de projetos, clientes, fornecedores, produtos e departamentos
- **Datas**: Data da reunião/documento, prazos, datas de ações e marcos
- **Pessoas**: Participantes da reunião, responsáveis por ações, assinantes
- **Decisões**: Resoluções tomadas, aprovações, rejeições e mudanças de direcionamento
- **Ações e Responsabilidades**: Tarefas atribuídas, responsáveis, prazos associados
- **Riscos e Oportunidades**: Riscos identificados, impactos e oportunidades de negócio
- **Temas e Categorias**: Área funcional (vendas, TI, operações), tipo de documento, tema principal
- **Métricas e Números**: Valores de vendas, metas, percentuais, quantidades
- **Relacionamentos**: Conexões entre reuniões, documentos, projetos e decisões anteriores
- **Confidencialidade**: Nível de sensibilidade dos dados (público, interno, confidencial)
```

---

## 1.4 Estratégia de classificação inicial

Como você classificaria os documentos sem depender de subpastas dentro de `raw/`?

**Sua resposta:**

```md
Classificação por metadados em S3 e DynamoDB, sem depender de subpastas:

1. **Tipo de Arquivo** (extensão):
   - PDF: ata_reuniao_vendas_sa.pdf
   - Imagem/PNG: ata_resultados_vendas_novos_dados.png
   - Dados Estruturados: vendas_sa_dados_ficticios_laboratorio.csv

2. **Categoria Funcional** (extraída ou inferida):
   - Documentos de Vendas (atas e dados de oportunidades)
   - Documentos de Resultados (ata de resultados)

3. **Tipo de Processamento Necessário**:
   - Com OCR: ata_resultados_vendas_novos_dados.png
   - Extração de Texto: ata_reuniao_vendas_sa.pdf
   - Parsing Estruturado: vendas_sa_dados_ficticios_laboratorio.csv

4. **Metadados Adicionais** (armazenados em DynamoDB):
   - Nome original do arquivo
   - Data de ingestão
   - Status de processamento (pendente, processado, com erro)
   - Tamanho do arquivo
   - Caminho S3 do original e processado
   - Confiança do OCR/Extração (quando aplicável)
   - Nível de confidencialidade (padrão: interno)

5. **Tags S3**:
   - `tipo-documento`: ata, dados, relatório
   - `departamento`: vendas
   - `ano`: 2024
   - `status`: raw, processado
```

---

# ✅ Quest 2: O Portal de Entrada na AWS

## 2.1 Armazenamento dos arquivos brutos

Explique como os arquivos da pasta `raw/` seriam enviados e armazenados na AWS.

Serviços que você pode considerar:

- Amazon S3
- AWS IAM
- AWS KMS
- Amazon S3 Versioning
- Amazon S3 Lifecycle

**Sua resposta:**

```md
Arquivos são enviados e armazenados seguindo este fluxo:

1. **Armazenamento em S3**:
   - Criar bucket S3: `corporativo-wiki-raw` na região apropriada
   - Estrutura no S3: `s3://corporativo-wiki-raw/raw/{arquivo}`
   - Manter nomes de arquivo originais para rastreabilidade

2. **Versionamento**:
   - Ativar S3 Versioning no bucket para manter histórico completo
   - Facilita auditoria e recuperação de versões anteriores

3. **Encriptação de Dados**:
   - Usar AWS KMS para encriptação Server-Side (SSE-KMS)
   - Criar customer-managed key no KMS por departamento/acesso
   - Encriptação em repouso e em trânsito (HTTPS obrigatório)

4. **Controle de Acesso**:
   - AWS IAM: Criar política específica com permissão apenas de leitura para lambda/step functions
   - Bloquear acesso público com S3 Block Public Access
   - Usar MFA Delete para maior segurança

5. **Lifecycle Policy**:
   - Raw files: Guardar em STANDARD por 30 dias
   - Depois mover para STANDARD-IA ou GLACIER (backup de longo prazo)
   - Documentos processados com sucesso podem ter retenção menor

6. **Metadados S3**:
   - Adicionar object tags (tipo, departamento, ano, confidencialidade)
   - Adicionar user-defined metadata (data de ingestão, versão, origem)
```

---

## 2.2 Preservação dos arquivos originais

Explique como garantir que os arquivos originais sejam mantidos intactos e rastreáveis.

**Sua resposta:**

```md
Estratégia de preservação e rastreabilidade:

1. **Separação Física de Buckets S3**:
   - Bucket RAW: Apenas para arquivos originais (read-only após ingestão)
   - Bucket PROCESSED: Para arquivos processados e extratos de texto
   - Isso impede alteração acidental de originais

2. **Registro de Integridade**:
   - Calcular hash SHA-256 de cada arquivo na ingestão
   - Armazenar hash em DynamoDB com metadados
   - Validar integridade periodicamente usando Lambda

3. **S3 Object Lock**:
   - Ativar modo compliance em arquivos críticos
   - Impede deletamento ou sobrescrita, mesmo por administrador
   - Retenção mínima de 7 anos (ou conforme política corporativa)

4. **Rastreabilidade Completa**:
   - AWS CloudTrail: Registra todas as operações S3 (get, put, delete)
   - DynamoDB: Tabela `DocumentMetadata` com histórico
   - Campos: arquivo_id, nome_original, caminho_s3, hash_sha256, data_ingestao, usuario_upload, status_processamento
   - Cada versão possui um document_version_id único

5. **Auditoria de Acesso**:
   - CloudTrail registra quem, quando e o quê acessou cada arquivo
   - Integração com CloudWatch Logs para alertas
   - Relatórios mensais de acesso aos arquivos

6. **Backup Redundante**:
   - Replicação cruzada para região secondary usando S3 Replication Rules
   - Backup diário em Glacier para recuperação de desastres
```

---

## 2.3 Extração de texto dos documentos

Explique como cada tipo de arquivo seria processado.

Considere:

- PDFs escaneados;
- Imagens;
- PDFs digitais;
- Arquivos `.txt`;
- Arquivos `.docx`;
- Arquivos `.md`.

Serviços que você pode considerar:

- Amazon Textract
- AWS Lambda
- AWS Step Functions
- Amazon S3
- Amazon CloudWatch

**Sua resposta:**

```md
Pipeline de extração com AWS Step Functions orquestrando o processamento:

1. **Trigger - EventBridge/S3 Notification**:
   - Quando arquivo aparece em s3://corporativo-wiki-raw/raw/
   - Inicia execução de StateMachine no Step Functions
   - Registra início em DynamoDB com status: "PROCESSANDO"

2. **Roteamento por Tipo de Arquivo**:
   Arquivo → Lambda (Detector de Tipo) → Tipo identificado

3. **Processamento por Tipo**:
   
   **PDFs Digitais (com camada de texto)**:
   - Lambda simples: Usar boto3 com pdfplumber ou PyPDF2
   - Extrair texto, tabelas e metadados
   - Saída em S3 processado + DynamoDB

   **PDFs Escaneados + Imagens (PNG, JPG)**:
   - Amazon Textract: Enviar arquivo via boto3
   - Uso de Async Textract para documentos > 5MB
   - Receber JSON com blocos extraídos (palavras, linhas, tabelas)
   - Reconstituir estrutura: cabeçalho, corpo, tabelas, assinatura
   - Textract fornece confiança (confidence score) para validação

   **Arquivos CSV/Dados Estruturados**:
   - Lambda: Parse CSV com pandas
   - Validação de schema esperado (19 colunas)
   - Conversão para JSON estruturado
   - Registro de metadados (linhas, colunas, tipos de dados)

   **DOCX/Markdown**:
   - Lambda: Usar python-docx para DOCX
   - Extrair texto, mantendo estrutura (headings, tabelas)
   - Markdown: parsing simples

4. **Pós-processamento**:
   - Limpeza de texto: Remover caracteres especiais, normalizar espaços
   - Validação: Verificar se texto não vazio e confiança > 70%
   - Chunking: Dividir textos em trechos (~500 tokens) para embedding
   - Armazenar processado em S3 em estrutura JSON: {conteudo, metadados, chunks, confianca}

5. **Tratamento de Erros Integrado**:
   - Textract falha → Retry automático (2x)
   - Se PDF muito grande → Dividir em páginas, processar cada uma
   - Se OCR falha → Registro em DynamoDB com flag REQUER_REVISAO_MANUAL

6. **Monitoramento**:
   - CloudWatch: Logs de cada lambda, métricas de latência
   - SNS: Notificação para fila manual se falha após retries
```

---

## 2.4 Tratamento de falhas

Explique como sua solução identificaria e registraria erros de processamento.

**Sua resposta:**

```md
Sistema de tratamento de falhas com múltiplas camadas:

1. **Camada Step Functions**:
   - Definir retry policy em cada task
   - Textract com backoff exponencial: 2 tentativas, delay 5-30s
   - Lambda com timeout de 10 minutos
   - Catch para exceções: roteamento para Dead Letter Queue

2. **Registro de Erros em DynamoDB**:
   - Tabela `ProcessingErrors` com campos:
     - document_id (PK): arquivo_reuniao_vendas_sa.pdf
     - timestamp (SK): 2024-01-15T10:30:00Z
     - error_type: TEXTRACT_CONFIDENCE_LOW, PDF_PARSING_FAILED, OCR_TIMEOUT
     - error_message: descrição completa do erro
     - stack_trace: log técnico
     - retry_count: número de tentativas
     - status: FALHA_PERMANENTE, AGUARDANDO_RETENTATIVA, ESCALADO_PARA_HUMANO
     - responsavel: usuário que deve revisar manualmente

3. **CloudWatch Logging**:
   - Cada Lambda publica logs estruturados em JSON
   - Log Group: `/aws/lambda/wiki-corporativa-{funcao}`
   - Filtros de métrica para contabilizar erros por tipo
   - Alarme: Se taxa de erro > 10% em 5 min → Notificação SNS

4. **Escalação Automática**:
   - Erros de OCR com confiança baixa (< 60%)
   - Textract timeout em arquivo grande
   - PDF com caracteres ilegíveis
   - → Criar tarefa em fila SQS `DocumentosComErro` para revisão humana

5. **Mecanismo de Retry Inteligente**:
   - Falha no Textract → Tentar 2x com backoff
   - Se persistir → Dividir PDF em páginas menores
   - Se ainda falhar → Armazenar em bucket `revisar-manualmente`
   - Notificar time de operações para ação manual

6. **Relatório de Status**:
   - Dashboard em CloudWatch mostrando:
     - Total de arquivos processados
     - Taxa de sucesso (%)
     - Distribuição de erros por tipo
     - Tempo médio de processamento
     - Documentos pendentes
   - Relatório diário via email para gerente

7. **Rastreabilidade Completa**:
   - Correlação ID (trace_id) linkando:
     - Arquivo original em S3
     - Tentativas de processamento
     - Erros registrados
     - Ações de correção
```

---

# ✅ Quest 3: A Relíquia dos Metadados

## 3.1 Padronização dos textos processados

Explique como os textos extraídos seriam limpos, normalizados e preparados para consulta.

**Sua resposta:**

```md
Pipeline de limpeza e normalização de texto:

1. **Limpeza Inicial (Lambda)**:
   - Remover caracteres de controle e especiais desnecessários
   - Remover espaços múltiplos, quebras de linha extras
   - Normalizar codificação Unicode (NFC)
   - Remover watermarks, números de página, cabeçalhos/rodapés

2. **Normalização Linguística**:
   - Converter para UTF-8 uniforme
   - Padronizar case: manter original, criar versão lowercase para busca
   - Remover acentos duplicados ou caracteres corruptos do OCR
   - Expandir abreviações comuns (Sr. → Senhor, etc.)

3. **Estruturação de Conteúdo**:
   - Identificar e manter seções estruturadas (data, participantes, agenda)
   - Separar corpo do documento de metadados implícitos
   - Marcar tabelas com delimitadores especiais
   - Preservar formatação importante (negrito, listas) em markdown

4. **Tokenização e Segmentação**:
   - Dividir texto em sentenças usando NLTK/spaCy
   - Criar chunks de ~500 tokens com overlap de 50 tokens
   - Associar cada chunk a: página, seção, posição no documento
   - Metadado de contexto: início_original_posição, fim_original_posição

5. **Desduplicação**:
   - Calcular hash MD5 de cada chunk
   - Comparar com chunks já processados
   - Evitar armazenar duplicatas no banco vetorial
   - Registrar documento_id referenciado para chunks duplicados

6. **Validação de Qualidade**:
   - Verificar comprimento mínimo (> 10 palavras)
   - Validar OCR confidence (Textract score)
   - Detectar idioma (deve ser português)
   - Flag: QUALIDADE_BAIXA se confiança < 70%

7. **Enriquecimento**:
   - Remover PII (informações pessoais sensíveis):
     - Mascarar CPF, CNPJ, emails, telefones
     - Substituir nomes de pessoas por placeholder
   - Identificar entidades nomeadas (usando EntityRecognition)
   - Adicionar metadados de sensibilidade

8. **Armazenamento Padronizado**:
   - Salvar em S3 em formato JSON estruturado com campos: document_id, chunk_id, original_text, cleaned_text, tokens, page_number, section, confidence, quality_flag, entities, markdown_text
```

---

## 3.2 Metadados propostos

Defina quais metadados você extrairia de cada documento.

| Metadado | Por que ele é importante? |
|---|---|
| Nome do documento | Identificação única e rastreabilidade do arquivo original |
| Tipo do documento | Permite filtrar buscas por categoria (ata, relatório, dados) |
| Data identificada | Crucial para contexto temporal e linha do tempo de decisões |
| Tema principal | Classificação por área funcional facilitando navegação |
| Participantes | Rastreamento de quem esteve envolvido em decisões e ações |
| Decisões tomadas | Permite buscar "quais decisões foram tomadas sobre X?" |
| Responsáveis | Identifica proprietários de ações para follow-up |
| Próximos passos | Rastreia pendências, deadlines e ações abertas |
| Nível de confidencialidade | Controle de acesso e segurança da informação |
| Caminho do arquivo original | Link direto ao documento fonte para auditoria |
| Hash SHA-256 | Integridade e deduplicação |
| Data de ingestão | Rastreamento quando foi inserido na wiki |
| Última atualização | Quando o documento foi processado ou revisado |
| Confiança de extração | Score do OCR/Textract para validação de qualidade |
| Departamento | Organização por área funcional |
| Palavras-chave | Tags para busca rápida |
| Resumo automático | Síntese gerada por IA de 1-2 sentenças |
| Riscos identificados | Extração de riscos mencionados |
| Métricas e números | KPIs e valores importantes encontrados |
| Status de processamento | Se foi processado com sucesso ou aguarda revisão |

---

## 3.3 Uso de IA para enriquecimento dos documentos

Explique como o Amazon Bedrock poderia ajudar a identificar temas, decisões, responsáveis, pendências e resumos dos documentos.

**Sua resposta:**

```md
Integração do Amazon Bedrock para enriquecimento inteligente:

1. **Modelo Selecionado**:
   - Usar Claude 3 via Amazon Bedrock (API serverless, sem gerenciar infraestrutura)
   - Alternativa: Llama 2 ou Mistral para custo reduzido
   - Batch processing: Aproveitar Bedrock Batch API para economia de 50%

2. **Pipeline de Enriquecimento**:
   Texto Limpo → Lambda → Amazon Bedrock → Metadados Extraídos → DynamoDB

3. **Prompts Estruturados** (Few-shot learning):
   
   **Identificação de Decisões**:
   - Prompt: "Liste todas as decisões tomadas neste documento. Formato: ['decisão', 'data', 'responsável']"
   - Bedrock analisa contexto e identifica com high accuracy

   **Extração de Responsáveis**:
   - Prompt: "Quem é responsável por cada ação? Retorne: [{'ação': '', 'responsável': '', 'prazo': ''}]"
   - Modelo vincula pessoas a ações e datas

   **Próximos Passos/Pendências**:
   - Prompt: "Quais são os próximos passos e ações abertas? Retorne JSON com ações e prazos"
   - Identifica tarefas não concluídas

   **Tema e Categoria**:
   - Prompt: "Qual é o tema principal? Classifique como: [vendas, TI, operações, RH, financeiro, outro]"
   - Associa documento a categoria para filtros

   **Resumo Executivo**:
   - Prompt: "Resuma em 2-3 sentenças o conteúdo e decisões principais"
   - Gera resumo de 100 tokens para preview rápido

4. **Extração de Entidades Nomeadas** (NER):
   - Bedrock identifica: pessoas, organizações, projetos, datas, números
   - Estruturado em JSON com tipos de entidade
   - Facilita linking entre documentos (ex: mesmo projeto em várias atas)

5. **Análise de Sentimento e Contexto**:
   - Detecta tom (urgente, normal, informativo)
   - Identifica riscos mencionados implicitamente
   - Flag para escalação se: "palavras risco, urgência, problema crítico"

6. **Armazenamento dos Metadados Extraídos**:
   - DynamoDB tabela `DocumentMetadataEnriched` com: document_id, decisoes, proximos_passos, tema, resumo, entidades, riscos, confianca_ia, timestamp_enriquecimento

7. **Validação Humana Opcional**:
   - Se confiança IA < 0.85 → Incluir em fila de revisão
   - Interface web simples para confirmar/corrigir metadados
   - Feedback loop: corrigir modelo com dados validados

8. **Otimização de Custo**:
   - Usar Bedrock On-Demand para documentos urgentes
   - Usar Bedrock Provisioned Units para volume > 1000 docs/dia
   - Cachear prompts do sistema para reduzir tokens
```

---

## 3.4 Armazenamento dos metadados

Explique onde os metadados seriam armazenados e como seriam conectados aos documentos originais.

Serviços que você pode considerar:

- Amazon S3
- Amazon DynamoDB
- AWS Glue Data Catalog
- Amazon Bedrock Knowledge Bases

**Sua resposta:**

```md
Arquitetura de armazenamento de metadados em múltiplas camadas:

1. **Camada Principal: Amazon DynamoDB**:
   - **Tabela `DocumentMetadata`** (índices de busca rápida):
     - PK: `document_id` (ata_reuniao_vendas_sa)
     - SK: `version` (v1, v2, etc.)
     - Atributos: nome, tipo, data, departamento, status, caminho_s3
     - GSI1: `departamento#data` (buscar por departamento)
     - GSI2: `tema#status` (buscar por tema)
     - TTL: opcional, para deletar metadados de docs expirados

   - **Tabela `DocumentEnrichedMetadata`** (resultado do Bedrock):
     - PK: `document_id`, SK: `enrichment_version`
     - Atributos: resumo, decisoes (list), responsaveis (set), riscos, confianca_ia
     - Timestamps de última atualização

   - **Tabela `ProcessingLog`** (auditoria completa):
     - PK: `document_id`, SK: `timestamp`
     - Registra cada etapa: upload, textract, bedrock, indexação
     - Facilita debugging e compliance

2. **Conectar Metadados ao Documento Original**:
   - Campo `s3_path` em DynamoDB aponta para S3:
     s3://corporativo-wiki-raw/raw/ata_reuniao_vendas_sa.pdf
   - Versionamento S3: todos os acessos passam por version_id
   - Permitir recuperação exata da versão processada

3. **Camada Secundária: AWS Glue Data Catalog**:
   - Registrar tabelas lógicas: documentos, metadados, chunks
   - Facilita descoberta de dados corporativos
   - Integração com AWS Athena para queries SQL

4. **Camada Terciária: Amazon Bedrock Knowledge Bases**:
   - Armazena vetores de embedding dos chunks
   - Associa chunk_id → document_id → S3 original
   - Permite busca semântica rápida
   - Retorna documento fonte com confiança

5. **Arquivo de Índice em S3 (JSON Lines)**:
   - Backup e auditoria em `s3://corporativo-wiki-raw/indexes/`
   - Arquivo: `documents_index_2024_01_15.jsonl`
   - Cada linha: {document_id, s3_path, hash, timestamp, status, metadados}
   - Permite reconstruir índices se DynamoDB corrupto

6. **Relacionamento Entre Metadados**:
   - DynamoDB Query: Encontrar documento pelo ID
   - Glue Data Catalog: Descobrir documentos por tema/departamento
   - Bedrock KB: Buscar por semântica (pergunta → chunks → documentos)
   - CloudTrail: Auditar acessos

7. **Exemplo de Fluxo de Consulta**:
   Usuário pergunta: "Quais decisões foram tomadas sobre vendas?"
   → Bedrock converte em embedding
   → Bedrock KB retorna top chunks + document_ids
   → Lambda busca DocumentMetadata + DocumentEnrichedMetadata
   → Compõe resposta com: resumo, decisões, responsáveis, link S3 original
   → CloudTrail registra: user_id, query, documentos_acessados, timestamp

8. **Sincronização e Consistência**:
   - DynamoDB Streams: Dispara lambda quando metadados mudam
   - Atualiza Bedrock KB incrementalmente
   - Glue Crawler executa a cada 6h para sincronizar schema
```

---

# ✅ Quest 4: O Oráculo da Wiki Inteligente

## 4.1 Estratégia de indexação

Explique como os documentos seriam divididos em trechos menores e preparados para busca semântica.

**Sua resposta:**

```md
Estratégia de chunking e indexação para busca semântica:

1. **Divisão em Chunks**:
   - Tamanho base: ~500 tokens (aproximadamente 1500 caracteres)
   - Overlap: 50 tokens entre chunks consecutivos (preserva contexto)
   - Estratégia: Quebrar por sentenças, não no meio de palavras
   - Usar bibliotecas: tiktoken para contar tokens exatamente

2. **Preservação de Contexto Hierárquico**:
   - Manter estrutura: Documento → Seção → Parágrafo → Chunk
   - Adicionar metadados a cada chunk:
     - `parent_doc_id`: ata_reuniao_vendas_sa
     - `section`: "Decisões", "Próximos Passos", etc.
     - `position`: pagina 2, parágrafo 3
     - `source_start_pos`: byte 1523
     - `source_end_pos`: byte 2045

3. **Tipos de Chunks**:
   
   **Chunks Textuais** (padrão):
   - Quebra por tamanho e sentenças
   - Mantém fluxo natural de leitura
   
   **Chunks Estruturados** (tabelas do CSV/Textract):
   - Tabelas convertidas para texto estruturado
   - Cada linha é um chunk com formato estruturado
   
   **Chunks de Resumo** (metadata chunks):
   - Resumo do documento inteiro
   - Metadados estruturados em linguagem natural
   - Facilita busca por "quais atas mencionam segurança?"

4. **Pipeline de Indexação (Lambda + Step Functions)**:
   Documento Processado (S3) → Lambda: Carregar e chunkar → Para cada chunk: Calcular embedding (Bedrock) → Armazenar em Bedrock Knowledge Base → Registrar índice em DynamoDB → Completar indexação

5. **Deduplicação de Chunks**:
   - Calcular hash SHA-256 de cada chunk
   - Verificar se hash já existe em DynamoDB (tabela ChunkIndex)
   - Se duplicado: referenciar chunk_id original, não reindexar
   - Economiza custo de embedding e espaço de armazenamento

6. **Versionamento de Índices**:
   - Quando documento é atualizado:
     - Chunks antigos: marcar como `deprecated`
     - Novos chunks: versionar com `chunk_version: v2`
     - Bedrock KB: suporta múltiplas versões de documentos
   - Permite pesquisar versão específica se necessário

7. **Otimização de Busca**:
   - Índices secundários em DynamoDB:
     - Por tema: topic#sector (vendas#tecnologia)
     - Por data: document_date#sector
     - Por confiança: confidence_score#status
   - Permite pré-filtrar antes de embeddings (economiza tokens)

8. **Monitoramento de Qualidade**:
   - CloudWatch métrica: `ChunksIndexed` por tipo de documento
   - Alerta se taxa de duplicação > 20% (indica problema)
   - Validar token count consistente (não chunks muito pequenos)
```

---

## 4.2 Busca semântica e base vetorial

Explique como embeddings seriam gerados e onde seriam armazenados.

Serviços que você pode considerar:

- Amazon Bedrock Knowledge Bases
- Amazon OpenSearch Serverless
- Amazon Aurora PostgreSQL com pgvector
- Amazon S3 Vectors
- Modelos de embeddings no Amazon Bedrock

**Sua resposta:**

```md
Arquitetura de embeddings e busca semântica:

1. **Escolha do Modelo de Embedding**:
   - **Recomendado**: Bedrock Knowledge Bases (gerenciado, simples)
   - Alternativa técnica: OpenSearch Serverless (mais controle)
   - Modelo selecionado: Embeddings Titan Text (disponível em Bedrock)
   - Dimensionalidade: 1536 (tamanho padrão)
   - Suporta português naturalmente
   - Otimizado para RAG (Retrieval-Augmented Generation)

2. **Geração de Embeddings**:
   Chunk de Texto → Bedrock Embeddings API → Vetor 1536-D
   - Bedrock recebe chunk (máx 2000 tokens)
   - Retorna embedding (vetor numérico)
   - Latência: ~100ms por chunk
   - Batch processing: até 100 chunks por chamada

3. **Armazenamento de Vetores - Amazon Bedrock Knowledge Bases**:
   - **Serviço gerenciado** de vetores + RAG
   - Suporta múltiplas fontes de dados (Documentos em S3, Texto estruturado JSON/CSV)
   - Configuração: Create Knowledge Base com Name: wiki-corporativa-kb, Model: Bedrock Embeddings Titan Text, Storage: Managed Vector Store (serverless), Capacity: Auto-scaling
   - Metadados armazenados com cada vetor: embedding, chunk_id, document_id, document_type, department, date, section, confidence, source_page, text_preview

4. **Alternativa: OpenSearch Serverless** (mais flexibilidade):
   - Se precisar busca híbrida (keyword + semântica)
   - OpenSearch mantém índices Lucene + vetores
   - Chunk → Lambda → Extrai keywords → OpenSearch Ingestion → Bedrock Embeddings → OpenSearch Serverless (índice híbrido)
   - Query: Busca por palavra-chave + similaridade semântica

5. **Ciclo de Vida do Embedding**:
   - **Criação**: Quando chunk é criado (Lambda → Bedrock → KB)
   - **Atualização**: Quando chunk é revisado (recalcular embedding)
   - **Deprecação**: Quando chunk é marcado como obsoleto
   - **Limpeza**: Manter últimos 2 anos, arquivar histórico em Glacier

6. **Otimização de Custos**:
   - Bedrock Knowledge Bases: modelo de preço por 1K embeddings
   - Estratégia: Cachear embeddings em S3, Reindexar apenas novos/atualizados, Usar batch API (desconto 50%)
   - Estimativa: 1000 documentos = ~50K chunks = $0.50 por mês

7. **Busca Semântica - Fluxo Completo**:
   Pergunta do Usuário → Bedrock Embeddings: pergunta → vetor 1536-D → Bedrock KB: buscar top-k=10 chunks similares → Retorna chunks com score similaridade (0-1) → Filtro: confiança > 0.7 E tipo_documento = "ata" → Resultado: chunks ordenados por relevância

8. **Qualidade e Monitoramento**:
   - Métrica: Mean Reciprocal Rank (MRR) das buscas
   - CloudWatch: registrar queries e relevância feedback
   - Alarme se MRR cair abaixo de 0.7 (indica qualidade ruim)
   - Quarterly: avaliar embedding model e considerar upgrade
```

---

## 4.3 Geração de respostas com IA

Explique como a Wiki responderia perguntas em linguagem natural com base nos documentos originais.

Considere explicar:

- Como a pergunta do usuário seria recebida;
- Como os trechos relevantes seriam recuperados;
- Como o Amazon Bedrock geraria a resposta;
- Como a resposta indicaria as fontes utilizadas.

**Sua resposta:**

```md
Fluxo completo de pergunta-resposta com RAG (Retrieval-Augmented Generation):

1. **Recebimento da Pergunta**:
   - Usuário acessa interface (web, chat, API)
   - Exemplo: "Quais foram as decisões sobre o projeto de vendas em janeiro?"
   - Frontend envia pergunta via HTTPS para API Gateway

2. **Roteamento e Processamento**:
   API Gateway → Lambda (WikiController)
   ├─ Autenticação: Cognito verifica token JWT
   ├─ Autorização: IAM valida permissões por departamento
   ├─ Auditoria: CloudTrail registra pergunta e user_id
   └─ Validação: Sanitizar entrada, verificar comprimento

3. **Recuperação de Documentos Relevantes** (Retrieval):
   Pergunta: "Quais foram as decisões sobre o projeto de vendas em janeiro?"
   
   A. Bedrock Knowledge Bases:
      - Gerar embedding da pergunta
      - Buscar top-10 chunks mais similares
      - Retorna chunks com score: 0.95, 0.92, 0.88, ...
   
   B. Filtro Adicional (Lambda):
      - Manter apenas chunks de documentos em janeiro
      - Tema = "vendas"
      - Confiança extração > 0.7
      - Resultado: 5-7 chunks altamente relevantes
   
   C. Estrutura de cada chunk retornado: {chunk_id, document_id, document_name, text, similarity_score, source_page, extraction_confidence}

4. **Construção do Prompt para Bedrock**:
   System Prompt: "Você é um assistente especializado em wiki corporativa. Resuma decisões, ações e riscos com base APENAS em documentos fornecidos. Sempre cite as fontes (documento e data)."
   
   User Message: Pergunta + Contexto dos documentos (chunks com páginas e informações)

5. **Geração de Resposta por Bedrock**:
   - Modelo: Claude 3 Sonnet (bom equilíbrio custo-qualidade)
   - Temperatura: 0.3 (menos criativo, mais preciso)
   - Max tokens: 1000 (resposta concisa)
   - Latência esperada: 2-5 segundos

6. **Formatação da Resposta com Rastreabilidade**:
   Retorna JSON com: answer, sources, chunks_used, confidence, latency_ms, timestamp, user_id
   - answer: Resposta em português
   - sources: Array com documento_id, document_name, date, s3_link, relevance_score
   - chunks_used: Quais chunks foram usados na resposta
   - confidence: Score de qualidade (0-1)

7. **Feedback e Melhoria Contínua**:
   - Usuário pode avaliar resposta: "Útil?" (sim/não/parcial)
   - Ratings alimentam modelo de ranking em DynamoDB
   - Bedrock Fine-tuning: usar feedback para melhorar prompts
   - Análise: se muitas respostas rated baixas → revisar chunks/modelo

8. **Tratamento de Falhas**:
   - Se nenhum chunk relevante: "Não encontrei documentos específicos..."
   - Se timeout Bedrock (>10s): retornar chunks diretos sem IA
   - Se erro autenticação: "Você não tem permissão para acessar esse departamento"
```

---

## 4.4 Interface de consulta

Proponha como os usuários acessariam essa Wiki Inteligente.

Serviços que você pode considerar:

- Amazon Q Business
- AWS Amplify
- Amazon API Gateway
- AWS Lambda
- Amazon Cognito

**Sua resposta:**

```md
Arquitetura de interface e acesso à Wiki Inteligente:

1. **Camada de Autenticação - Amazon Cognito**:
   - User Pool: armazenar usuários corporativos
   - Integração com SSO (Active Directory ou similar)
   - MFA obrigatório para acesso sensível
   - Permissões por grupo: vendas, TI, operações, executivo
   - Gera JWT token válido por 1 hora

2. **Opção 1: Amazon Q Business** (Recomendado - Gerenciado):
   - Serviço nativo AWS para busca e chat em documentos
   - Integra Bedrock + Knowledge Bases automaticamente
   - Interface web pré-pronta (no-code)
   - Funcionalidades: Chat em linguagem natural, Busca com autocomplete, Recomendação de documentos relacionados, Histórico de conversas por usuário
   
   Configuração:
   Amazon Q Business → Cognito (autenticação) → Bedrock KB (recuperação) → Bedrock Claude (geração) → DynamoDB (metadados)

3. **Opção 2: Web App Customizado (AWS Amplify)**:
   - Frontend: React/Vue.js com Amplify UI Components
   - Backend: API Gateway + Lambda
   - Hospedagem: Amplify Hosting (CI/CD automático)
   
   Arquitetura:
   Usuário → Amplify Web App (React) → CloudFront (CDN) → API Gateway → Lambda (Query Handler) → Bedrock KB + Bedrock Claude → DynamoDB → CloudTrail

4. **Interface de Usuário - Funcionalidades**:
   
   **A. Busca Simples**:
   - Search bar com filtros laterais: Por tema, departamento, data, confiança
   - Resultados: Cards com resumo + link para documento
   
   **B. Chat Conversacional**:
   - Seguimento: "E quem foi responsável?"
   - Contexto mantido na sessão (5 últimas perguntas)
   - Respostas com citações em tempo real
   
   **C. Dashboard de Insights**:
   - Cards: Total documentos, última atualização, temas principais
   - Gráficos: Documentos por departamento/tema/mês
   - Alertas: "3 ações abertas vencidas em vendas"
   
   **D. Explorador de Documentos**:
   - Timeline: Visualizar atas cronologicamente
   - Árvore: Navegar por tema/departamento
   - Metadados: Mostra decisões, responsáveis, riscos

5. **Controle de Acesso por Interface**:
   - Executivo: acesso a todos os documentos
   - Gerente Vendas: apenas documentos de vendas
   - SDR: apenas resumos (sem detalhes estratégicos)
   - Implementado em Lambda (antes de retornar conteúdo)

6. **Mobile - Acesso via App**:
   - AWS Amplify Mobile SDK (iOS/Android)
   - Interface simplificada: chat only
   - Sincronização offline (últimas 20 conversas)
   - Notificações: quando ação atribuída a ele está vencida

7. **API para Integração**:
   - REST API via API Gateway com endpoints para query
   - Integração com Slack, Teams via webhooks

8. **Performance e Escalabilidade**:
   - CloudFront: caching de assets (images, JS, CSS)
   - Lambda: auto-scaling (0-1000 concurrent)
   - DynamoDB: auto-scaling de capacidade
   - SLA: 99.9% uptime, < 5s resposta P95

9. **Analytics e Monitoramento**:
   - CloudWatch Dashboard: Queries por hora/tema/usuário, Tempo médio de resposta, Taxa de erro, Popularidade de documentos
   - Amplitude/Segment: telemetria de user behavior
```

---

## 4.5 Segurança, auditoria e monitoramento

Explique como controlar acesso, proteger dados, auditar consultas e monitorar custos, erros e qualidade das respostas.

Serviços que você pode considerar:

- AWS IAM
- AWS KMS
- Amazon Cognito
- AWS CloudTrail
- Amazon CloudWatch
- Amazon Macie
- AWS Cost Explorer

**Sua resposta:**

```md
Estratégia integrada de segurança, auditoria e monitoramento:

1. **Controle de Acesso - AWS IAM + Cognito**:
   
   **Identidade**:
   - Cognito User Pool: SSO com Active Directory
   - Grupos: vendas, ti, operacoes, executivo, auditoria
   - MFA: TOTP ou SMS (obrigatório para admin)
   
   **Autorização Granular**:
   Policy: DocumentAccessByDepartment com Effect: Allow, Action: bedrock:Invoke + s3:GetObject, Resource restrito por departamento
   
   **Aplicação em Lambda**:
   - Validar departamento do usuário
   - Filtrar documentos antes de retornar
   - Mascarar informações sensíveis

2. **Encriptação de Dados**:
   
   **Em Repouso**:
   - S3: SSE-KMS com customer-managed key
   - DynamoDB: encryption enabled
   - KMS: uma chave por departamento/classificação
   
   **Em Trânsito**:
   - HTTPS/TLS 1.3 obrigatório
   - API Gateway: WAF (Web Application Firewall)
   - Bedrock: comunicação criptografada
   
   **Gerenciamento de Chaves**:
   - KMS Key Rotation: automática a cada 1 ano
   - CloudTrail registra acesso às chaves
   - Acesso restrito a admin/auditoria

3. **PII e Dados Sensíveis**:
   
   **Identificação**:
   - Amazon Macie: scan automático de S3
   - Detecta: CPF, CNPJ, emails, telefones, cartão de crédito
   - Alertas em tempo real se PII encontrada
   
   **Tratamento**:
   - Mascarar antes de indexar em Bedrock KB
   - Manter mapping criptografado em DynamoDB
   - Auditoria: quem acessou PII e quando
   
   **Conformidade**:
   - LGPD: direito ao esquecimento (deletar documento)
   - LGPD: portabilidade (exportar dados do usuário)

4. **Auditoria Completa - AWS CloudTrail**:
   
   **Logging Configurado**:
   - Trail: registra todas as API calls
   - Multi-region: central e regional
   - S3: armazenar logs por 7 anos
   - CloudTrail Insights: detecta anomalias
   
   **Eventos Auditados**:
   - S3: GetObject (acessar documento), PutObject (novo doc)
   - Bedrock: InvokeModel (pergunta feita)
   - DynamoDB: GetItem (buscar metadados)
   - Cognito: UserAuthentication (login)
   - Lambda: Invocation (processos)
   - KMS: Decrypt (acesso a chave)
   
   Cada evento contém: userIdentity, eventTime, eventSource, sourceIPAddress, requestParameters, responseElements

5. **Monitoramento Contínuo - CloudWatch**:
   
   **Métricas Técnicas**:
   - Lambda: duration, errors, throttles
   - Bedrock: latency, token usage, model errors
   - DynamoDB: consumed capacity, throttles
   - S3: request count, 4xx/5xx errors
   - Alertas: se métrica sai do normal
   
   **Métricas de Negócio**:
   - Queries por hora/dia (trending up? down?)
   - Documentos acessados (uso real)
   - Temas mais buscados (insights)
   - Feedback de usuário (resposta útil: %)
   
   **Dashboards**:
   - Painel 1: Operacional (latency, errors, uptime)
   - Painel 2: Segurança (failed logins, unauthorized access)
   - Painel 3: Custos (spend por serviço, por usuário)
   - Painel 4: Negócio (queries, temas, documentos populares)
   
   **Alertas** (SNS):
   - Erro taxa > 1% → pager (urgente)
   - Acesso negado > 10 vezes → investigação
   - Bedrock latency > 10s → optimize query
   - Custo diário > orçamento → notificação

6. **Monitoramento de Custos - AWS Cost Explorer**:
   
   **Alocação**:
   - Tags: cost-center (vendas, ti, operacoes)
   - Cost Allocation: faturar por departamento
   - Budget: alerta se gastos excedem 10%
   
   **Otimização**:
   - S3: Lifecycle rules (arquivo antigo → Glacier)
   - Bedrock: uso de batch API (50% desconto)
   - Lambda: rightsizing (aumentar timeout reduz concorrência)
   - DynamoDB: avaliar on-demand vs provisioned
   
   **Relatórios**:
   - Monthly: breakdown por serviço/departamento
   - Forecast: projeção para próximo mês
   - Anomaly Detection: identifica picos de gasto

7. **Monitoramento de Qualidade - Feedback Loop**:
   
   **Métricas de IA**:
   - Relevance: usuário achou resposta útil? (sim/não/parcial)
   - Accuracy: comparar resposta com documento fonte (manual)
   - Hallucination: IA inventou informação? (audit sample)
   - Coverage: pergunta foi respondida ou caiu em "não sei"?
   
   **Dashboard de Qualidade**:
   - % respostas úteis: alvo 80%+
   - % alucinações: alvo < 5%
   - Coverage: alvo 90%+
   - Trending: melhorando ou piorando?
   
   **Ações se Qualidade Cai**:
   - Revisar prompts do Bedrock
   - Fine-tuning com dados feedback
   - Reindexar documentos
   - Investigar chunks de baixa qualidade
```

---

# 🧩 Arquitetura Final da Solução

Agora reúna tudo em uma visão única.

## 1. Visão geral

Explique em poucas linhas a ideia central da sua arquitetura.

**Sua resposta:**

```md
Solução de Wiki Corporativa Inteligente que transforma documentos heterogêneos (PDFs, imagens escaneadas, arquivos CSV) em uma base de conhecimento pesquisável e respondível via IA.

Fluxo central: (1) Ingestão segura em S3 com versionamento, (2) processamento automático via Textract + Lambda + Step Functions com tratamento de erros, (3) enriquecimento com Bedrock para extração de decisões/ações/riscos, (4) armazenamento de metadados em DynamoDB e vetores em Bedrock Knowledge Bases, (5) busca semântica via embedding, (6) geração de respostas com RAG, (7) interface via Amazon Q Business ou web app, (8) auditoria completa e controle de acesso por departamento.

Resultado: executivos, gerentes e operacional conseguem perguntar em linguagem natural ("Quais decisões sobre vendas foram tomadas em janeiro?") e receber resposta imediata com documentos de origem, responsáveis e próximos passos.
```

---

## 2. Serviços AWS utilizados

| Serviço AWS | Papel na solução |
|---|---|
| Amazon S3 | Armazenamento seguro de documentos raw e processados com versionamento, encryption KMS e versioning |
| Amazon Textract | Extração de texto de PDFs escaneados e imagens com OCR, detecção de tabelas e confidence scoring |
| Amazon Bedrock | Modelos de IA: Claude para geração de respostas, Embeddings Titan para vetorização, fine-tuning de prompts |
| Amazon Bedrock Knowledge Bases | Gerenciamento de base vetorial, retrieval semântico e RAG, integração com Bedrock e S3 |
| AWS Lambda | Processamento serverless: detector de tipo, extração de texto, chunking, enriquecimento com IA |
| AWS Step Functions | Orquestração de pipeline de processamento, retry policies, error handling, state management |
| Amazon DynamoDB | Armazenamento de metadados, índices de busca, histórico de processamento, feedback de usuário |
| Amazon Cognito | Autenticação com SSO, gerenciamento de usuários, grupos de acesso por departamento, MFA |
| Amazon API Gateway | Exposição de endpoints REST para queries, autenticação JWT, rate limiting, CORS |
| Amazon CloudWatch | Logs centralizados, métricas, dashboards, alertas de erro/performance/custo |
| AWS CloudTrail | Auditoria completa de todas as operações, compliance LGPD, rastreamento de acesso |
| AWS KMS | Encriptação de dados em repouso e chaves por departamento/classificação |
| Amazon Macie | Detecção automática de PII (CPF, CNPJ, emails), alertas de dados sensíveis |
| AWS Cost Explorer | Monitoramento de gastos, alocação por departamento, otimizações de custo |
| Amazon EventBridge | Trigger automático quando novo arquivo é enviado para S3, inicia Step Functions |
| AWS Glue | Data Catalog para descoberta de dados, metadados, schema evolution |
| Amazon OpenSearch Serverless | Alternativa: busca híbrida (keyword + semântica) se maior flexibilidade necessária |
| Amazon SNS | Notificações: erros críticos, ações vencidas, alertas de segurança |
| Amazon SQS | Fila de processamento para documentos com erro, retry automático |
| AWS IAM | Políticas de acesso granular, princípio do menor privilégio, auditoria de permissões |
| AWS Secrets Manager | Armazenamento seguro de credenciais de API, tokens, chaves privadas |
| Amazon Q Business | Interface gerenciada para chat em documentos, busca com IA |
| AWS Amplify | Hospedagem web, CI/CD, frontend React para interface customizada |
| CloudFront | CDN para cache de assets, reduz latência global |

---

## 3. Fluxo de dados de ponta a ponta

Descreva o caminho dos dados desde a pasta `raw/` até a Wiki Inteligente.

```md
Exemplo de estrutura:

1. Arquivos estão inicialmente na pasta raw/
2. Arquivos são enviados para o Amazon S3
3. Documentos escaneados passam pelo Amazon Textract
4. Arquivos digitais têm seus textos extraídos
5. Textos são limpos e padronizados
6. Metadados são extraídos
7. Conteúdos são indexados em uma base pesquisável
8. Usuário pesquisa na Wiki
9. IA responde com base nos documentos originais
```

**Sua resposta:**

```md
Fluxo detalhado de ponta a ponta (54 etapas):

### FASE 1: INGESTÃO E ARMAZENAMENTO
1. Arquivos estão na pasta corporativa `raw/` (local)
2. Usuário usa AWS CLI/SDK ou console S3 para fazer upload
3. EventBridge dispara quando arquivo chega em s3://corporativo-wiki-raw/raw/
4. CloudTrail registra: quem, quando, arquivo, IP
5. S3 Versioning mantém histórico de todas as versões
6. S3 Object Lock protege em modo compliance (7 anos)
7. Arquivo é criptografado com KMS customer key
8. Metadados iniciais criados em DynamoDB (nome, tamanho, hash SHA256, data_ingestao)

### FASE 2: PROCESSAMENTO (Step Functions StateMachine)
9. EventBridge → Lambda DetectorDeTipo: identifica extensão (PDF/PNG/CSV)
10. Step Functions roteia para função apropriada:
    - CSV → Lambda ParseCSV (pandas) → JSON estruturado
    - PDF digital → Lambda ExtractPDF (PyPDF2) → texto
    - PDF escaneado + PNG → Amazon Textract API → JSON com blocos/tabelas/confiança
11. Se Textract falha → Retry 2x com backoff exponencial
12. Se persistir → Arquivo movido para `revisar-manualmente` bucket
13. Texto extraído é salvo temporariamente em S3 (staging area)
14. CloudWatch registra latência, tokens processados, confiança OCR

### FASE 3: LIMPEZA E NORMALIZAÇÃO
15. Lambda TextoCleaner: remove caracteres especiais, normaliza espaços, Unicode
16. Remove PII com regex: CPF, email (criptografa referências)
17. Detecta idioma (deve ser português)
18. Divide em chunks de ~500 tokens com overlap 50 tokens
19. Calcula hash MD5 de cada chunk → deduplica vs chunks anteriores
20. Qualidade: marca chunks com confidence < 70% para revisão
21. Textos limpos armazenados em S3 estruturados (JSON)

### FASE 4: ENRIQUECIMENTO COM IA
22. Lambda EnriquecedorComBedrock: prepara prompts estruturados
23. Para cada documento:
    - Prompt 1: "Identifique decisões, data, responsável"
    - Prompt 2: "Quais são próximos passos e prazos?"
    - Prompt 3: "Classifique tema: [vendas, ti, operacoes, ...]"
    - Prompt 4: "Resuma em 2-3 sentenças"
24. Bedrock Claude processa (temperatura 0.3 para precisão)
25. Metadados extraídos salvos em DocumentMetadataEnriched (DynamoDB)
26. Confiança IA registrada (alvo > 0.85)

### FASE 5: INDEXAÇÃO E EMBEDDINGS
27. Para cada chunk limpo: gerar embedding
28. Lambda → Bedrock Embeddings Titan → vetor 1536-D
29. Verificar deduplicação: hash chunk já indexado? → pular
30. Bedrock Knowledge Bases: armazena vetor + metadados
31. DynamoDB ChunkIndex: mapeia chunk_id → document_id → S3 path
32. Lambda indexação completa: atualiza DocumentMetadata com status INDEXADO

### FASE 6: INTERFACE E CONSULTA
33. Usuário acessa via Amazon Q Business ou web app (AWS Amplify)
34. Cognito autentica com SSO/token MFA
35. Pergunta: "Quais decisões sobre vendas em janeiro?"
36. API Gateway → Lambda QueryController
37. CloudTrail registra: usuário, pergunta, timestamp

### FASE 7: RECUPERAÇÃO (RAG - Retrieval)
38. Pergunta → Bedrock Embeddings → vetor 1536-D
39. Bedrock Knowledge Bases: buscar top-10 chunks similares
40. Lambda FiltroSemantico:
    - Manter apenas chunks com score > 0.75
    - Filtro: tema='vendas' AND data >= '2024-01-01'
    - Enriquecer com metadados (responsável, data, página)
41. Top 5-7 chunks retornados com preview

### FASE 8: GERAÇÃO DE RESPOSTA (RAG - Generation)
42. Lambda construtorDePrompt: compõe prompt do sistema + chunks + pergunta
43. Bedrock Claude (Sonnet): gera resposta em português
44. Latência típica: 2-5 segundos
45. Resposta inclui: resposta em linguagem natural, citações, links S3, confidence score

### FASE 9: RASTREABILIDADE E FEEDBACK
46. CloudTrail registra pergunta + resposta + chunks usados
47. Usuário avalia: "Resposta útil?" (sim/não/parcial)
48. DynamoDB QueryFeedback armazena rating
49. Se útil < 60%: adicionar à fila de melhoria (fine-tuning posterior)
50. Dashboard atualizado: respostas úteis trend

### FASE 10: AUDITORIA E MONITORAMENTO
51. CloudWatch coleta métricas: Queries/hora, tempo médio, taxa erro, chunks utilizados
52. Macie detecta PII → SNS alert
53. Taxa de sucesso Bedrock, custo de tokens
54. Alarmes: se erro > 1% ou latência > 10s → escalação
```

---

## 4. Diagrama textual da arquitetura

Crie um diagrama simples usando texto.

```md
Exemplo:

raw/ → Amazon S3 → Lambda/Step Functions → Textract → S3 Processado → Bedrock Knowledge Bases → Interface de Consulta → Usuário Final
```

**Sua resposta:**

```md
raw/ → S3 (raw, versionado, KMS) → EventBridge → Step Functions
         ↓
     Lambda Detector de Tipo
         ↓
    [CSV] [PDF Digital] [Imagem/Textract]
         ↓
    S3 Processed + DynamoDB Metadados
         ↓
    Lambda Cleaner + Chunking
         ↓
    Lambda Bedrock Enricher (decisões, responsáveis, tema, resumo)
         ↓
    Lambda Embeddings Generator → Bedrock Embeddings Titan (1536-D)
         ↓
    Bedrock Knowledge Bases (vetores + RAG) + DynamoDB ChunkIndex
         ↓
    [Amazon Q Business] ou [AWS Amplify React App]
         ↓
    Cognito (SSO, RBAC, MFA)
         ↓
    Usuário Final
         ↓
    API Gateway → Lambda Query Handler
         ↓
    Bedrock KB (retrieval) + Bedrock Claude (generation)
         ↓
    Resposta com fontes + CloudTrail (auditoria)
    
Paralelo: CloudWatch (monitoramento) | KMS (encriptação) | Macie (PII) | Cost Explorer (custos)
```

---

## 5. Riscos e limitações

Liste possíveis desafios da sua solução.

```md
Exemplo:
- Documentos ilegíveis podem prejudicar a extração de texto.
- OCR pode gerar erros em documentos com baixa qualidade.
- Custos podem aumentar conforme o volume de documentos.
- Metadados inferidos por IA podem precisar de validação humana.
- Respostas geradas por IA devem sempre referenciar documentos de origem.
```

**Sua resposta:**

```md
RISCOS TÉCNICOS:
1. Qualidade de OCR: Documentos com baixa resolução podem gerar erros
   - Mitigation: Macie + manual review para confiança < 70%
2. Alucinação de IA: Bedrock pode inventar informações
   - Mitigation: Sempre citar fontes, incluir confidence score
3. Latência de Embeddings: Indexação pode ficar lenta com volume grande
   - Mitigation: Batch processing, cache de embeddings
4. Escalabilidade DynamoDB: Hot partitions podem causar throttle
   - Mitigation: GSI bem desenhado, DAX cache
5. Custo de Bedrock: Tokens de embedding/geração acumulam
   - Mitigation: Batch API (50% desconto), cache de prompts

RISCOS DE DADOS E SEGURANÇA:
6. Dados Sensíveis Expostos: Documentos estratégicos vazados
   - Mitigation: KMS encryption, Cognito RBAC, Macie monitoring
7. PII Não Detectada: CPFs/emails expostos mesmo com Macie
   - Mitigation: Custom regex patterns, manual review antes de Bedrock
8. Integridade de Documentos: Arquivo deletado acidentalmente
   - Mitigation: Object Lock compliance mode, backup Glacier

RISCOS OPERACIONAIS:
9. Documentos Orphaned: Chunk indexado mas documento deletado
   - Mitigation: Validação periódica, purga automática
10. Falha de Processing: Documento fica preso em PROCESSANDO
    - Mitigation: Timeouts explícitos, retry strategy, DLQ
11. Desincronização de Índices: DynamoDB atualizado, Bedrock KB desatualizado
    - Mitigation: Transações, versioning explícito
12. Modelo de IA Degradado: AWS lança nova versão que quebra prompts
    - Mitigation: Versionamento de modelos, A/B testing

RISCOS DE NEGÓCIO:
13. Adoção Baixa: Usuários preferem buscar manualmente
    - Mitigation: Treinamento, interface intuitiva, quick wins
14. Confiança em Respostas de IA: Executivo toma decisão errada baseado em alucinação
    - Mitigation: Sempre exigir citação, disclaimer, audit trail
15. Compliance e Retenção: LGPD exige deletar documento, KB não permite deletar chunks
    - Mitigation: Manter mapping documento → chunks, deletar KB inteira se necessário

RISCOS DE CUSTOS:
16. Custos Inesperados: Textract + Bedrock escalam rapidamente
    - Mitigation: Budget alerts, cost estimation, rightsizing
17. Armazenamento Perpetual: S3 com versioning fica caro
    - Mitigation: Lifecycle policy agressivo, archival automático
```

---

## 6. Melhorias futuras

Descreva como a solução poderia evoluir.

```md
Exemplo:
- Criar uma interface web para consulta.
- Criar um chat interno para perguntas sobre atas.
- Adicionar controle de acesso por departamento.
- Criar dashboard de decisões e pendências.
- Gerar alertas automáticos sobre ações em aberto.
- Integrar com ferramentas corporativas.
```

**Sua resposta:**

```md
CURTO PRAZO (1-3 meses):
1. Interface Web Melhorada: Dashboard, filtros avançados, dark mode, mobile-first
2. Notificações Automáticas: Email quando novo documento favorito, ação vence, documento importante
3. Chat Slack/Teams: Bot inteligente com integração Lambda + API Gateway
4. Melhorias OCR: Textract Custom Queries, feedback loop de treinamento

MÉDIO PRAZO (3-6 meses):
5. Dashboard Executivo de Decisões: Timeline visual, KPI board, trend analysis, risk heatmap
6. Gerenciamento de Ações: Interface para criar/editar/fechar ações, escalação automática
7. Integração com Calendário Corporativo: Lembretes de reunião com documentos relacionados
8. Busca Facetada: Elasticsearch/OpenSearch para busca híbrida (keyword + semântica)

LONGO PRAZO (6-12 meses):
9. Integração Jira/Asana: Ações extraídas → criar issue automática
10. Análise de Redes: Gráficos de relacionamento (pessoas, documentos, decisões)
11. Fine-tuning de Modelo: Treinar Claude customizado para domínio corporativo
12. Predição de Riscos: ML model prever riscos em documentos futuros
13. Conformidade Dashboard: LGPD, retenção, auditoria automatizada
14. Integração PowerBI/Looker: Dashboard BI com dados de conhecimento
15. Chatbot Multilingue: Expandir para espanhol, inglês
16. Replicação Global: Bedrock KB em múltiplas regiões
17. Privacy-Preserving Analytics: Federated learning, differential privacy
18. Agentes Autônomos: Bedrock Agents com tool use (criar Jira, enviar email)
19. Knowledge Graph: Neo4j/Neptune para relacionamentos complexos
```

---

# 🧠 Checklist Final

Antes de entregar, confirme se sua solução responde:

- [x] Como transformar documentos escaneados em texto?
- [x] Como lidar com diferentes formatos dentro da mesma pasta `raw/`?
- [x] Como armazenar os documentos originais?
- [x] Como preservar a rastreabilidade entre resposta e documento fonte?
- [x] Como organizar metadados?
- [x] Como criar busca semântica?
- [x] Como usar Amazon Bedrock na solução?
- [x] Como proteger documentos sensíveis?
- [x] Como monitorar falhas?
- [x] Como a empresa usaria essa Wiki no dia a dia?

---

# 🏁 Conclusão

Escreva uma breve conclusão defendendo sua solução como se estivesse apresentando para uma liderança técnica ou de negócio.

**Sua resposta:**

```md
CONCLUSÃO - PROPOSTA DE WIKI CORPORATIVA INTELIGENTE COM AWS

A solução apresentada transforma o caos de documentos heterogêneos em um ativo corporativo estratégico: uma Wiki Inteligente, pesquisável, rastreável e segura.

PROBLEMA RESOLVIDO:
Antes: Executivos não conseguem responder "Quais decisões foram tomadas em janeiro?" sem procurar manualmente em pastas desordenadas.
Depois: Um clique retorna resposta em segundos com decisões, responsáveis, próximos passos e documentos de origem.

TECNOLOGIA AWS:
- Stack completo (Textract + Bedrock + Knowledge Bases = sem terceiros)
- Serverless e escalável (paga por uso, cresce automaticamente)
- Segurança enterprise (KMS, IAM, CloudTrail, Macie = compliance-ready)
- Custo previsível (modelo por API call, não VM 24/7)

IMPACTO NO NEGÓCIO:
- Velocidade: 10x mais rápida (informações em segundos)
- Conformidade: LGPD-ready com rastreamento completo
- Risco reduzido: Informações críticas não caem em esquecimento
- Eficiência: ~10 FTE × 10h/semana = $26k/mês economizados
- ROI: Custos de AWS pagos em 1-2 meses

INVESTIMENTO:
- Inicial: $30-50k setup
- Operacional: $5-10k/mês
- Payback: 1-2 meses

RECOMENDAÇÃO:
Aprovar para fase piloto com departamento de vendas (low risk, high impact).
Sucesso aqui → Rollout para toda empresa em 90 dias.
```

---

FIM DO DOCUMENTO
