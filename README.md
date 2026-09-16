# Phishing Analysis Lab

Laboratório pessoal de análise de phishing, criado para treinar e documentar o processo de triagem que um analista SOC N1/N2 executa ao receber uma denúncia: análise de cabeçalho de e-mail, verificação de autenticação (SPF/DKIM/DMARC), análise de URL e extração de IOCs.

Decidi montar este projeto porque queria entender na prática — não só na teoria — por que autenticação de e-mail "passar" nem sempre significa que o remetente é confiável, e como um analista chega da denúncia bruta até um relatório de incidente estruturado.

## Objetivo

Demonstrar, com casos reais, o fluxo completo de análise estática de phishing (sem execução de malware, sem sandbox dinâmico):

1. Receber a amostra (`.eml`)
2. Analisar cabeçalho (Received, From, Return-Path, Message-ID)
3. Verificar autenticação (SPF/DKIM/DMARC) e entender suas limitações reais
4. Analisar URLs/domínios (incluindo links encurtados)
5. Extrair IOCs
6. Documentar em relatório de incidente

## Ferramentas utilizadas

- **MXToolbox** — análise de cabeçalho e autenticação
- **VirusTotal** — reputação de domínio/URL/hash
- **urlscan.io** — sandbox de URL sem necessidade de acesso direto
- **CyberChef** — decodificação de conteúdo ofuscado
- **WHOIS / dig** — idade e resolução de domínio
- `exiftool`, `oletools`, `pdfid` — análise estática de anexos (quando presentes)

## Amostras

As amostras reais foram obtidas do [Phishing Pot](https://github.com/rf-peixoto/phishing_pot), repositório de e-mails de phishing coletados via honeypot para fins de pesquisa. Nenhum anexo executável é distribuído neste repositório — apenas hashes, quando aplicável.

## Case de exemplo — INC-001 (Apple iCloud)

E-mail se passando pela Apple, alegando limite de armazenamento do iCloud atingido. O ataque não tentou forjar o domínio da Apple — SPF/DKIM/DMARC do domínio real do atacante passaram normalmente, o que reforça um ponto importante: **autenticação válida não significa remetente legítimo**, apenas que o domínio remetente está autorizado a enviar por si mesmo. O corpo usava imagens ofuscadas atrás de links `bit.ly`, resolvendo para um domínio já reportado como phishing por múltiplos fornecedores no VirusTotal.

## Principais aprendizados

- SPF/DKIM/DMARC validam **posse de domínio**, não **legitimidade de identidade** — um atacante que registra o próprio domínio e configura autenticação corretamente passa limpo nos três
- A cadeia `Received:` se lê de baixo pra cima (ordem cronológica real) e costuma expor a origem real mesmo quando o `From:` está forjado
- Links encurtados devem ser expandidos sem visitar o destino diretamente (`curl -I` ou serviços de preview), evitando exposição de IP/user-agent ao atacante

## Aviso

Projeto estritamente educacional. Nenhum binário malicioso executável é distribuído neste repositório. Amostras reais vêm do Phishing Pot (honeypot de pesquisa); nenhum ataque foi conduzido contra terceiros.
