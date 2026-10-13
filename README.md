# Desafio DevSecOps — Gerenciador de Tarefas

## Sobre o Projeto
Este repositório faz parte do desafio prático do módulo de DevSecOps da ADA Tech. O objetivo foi configurar uma pipeline completa de integração contínua (CI/CD) com foco em segurança, automatizando a varredura de vulnerabilidades, a gestão de segredos e a assinatura digital de artefatos.

## Status da Implementação
- [x] Secrets Scanning com **Gitleaks**
- [x] SAST com **Semgrep**
- [x] SCA com **Grype**
- [x] Assinatura do artefato com **Cosign**
- [x] Deploy com **GitHub Pages**

---

## O que foi feito e corrigido

### 1. Detecção de Segredos (Gitleaks)
- **O problema:** Inicialmente, o Gitleaks encontrava chaves vazadas apenas quando rodado localmente, mas a esteira no GitHub Actions não as detectava porque os ficheiros com segredos não tinham sido enviados via `git push`. Além disso, havia uma senha de banco de dados exposta diretamente no código que não foi identificad nem localmente nem pelo github actions.
- **A correção:** Os segredos foram removidos do código-fonte e cadastrados na aba de *Secrets* do GitHub (`API_KEY` e `DB_PASSWORD`). No ficheiro da pipeline (`pipeline.yml`), foi utilizado o comando `sed` para injetar essas variáveis de ambiente dinamicamente no código logo antes do build.

### 2. Análise Estática de Código (Semgrep - SAST)
- **O problema:** O Semgrep encontrou 10 alertas de segurança. O principal ponto crítico no código foi o uso da função `eval()`, que concatenava o valor de um input do usuário, abrindo brecha para injeção de código (*code injection*). Também foram apontados avisos sobre o uso de tags mutáveis nas GitHub Actions (risco de *supply chain attack*).
- **A correção:** O código do ficheiro `src/script.js` foi limpo, substituindo a linha com `eval()` por um `console.log()` seguro. Na pipeline, a correção seria fixar as versões das ações.

### 3. Análise de Dependências (Grype - SCA)
- **O problema:** O Grype identificou diversas vulnerabilidades conhecidas (CVEs) em pacotes desatualizados como `axios`, `lodash`, `express` e `qs`. Também foi detectada uma vulnerabilidade na própria action `actions/download-artifact`.
- **A correção:** As dependências do projeto foram atualizadas para as suas versões mais recentes e seguras através de um container Node.js. No `pipeline.yml`, a versão da action de download foi atualizada para `actions/download-artifact@v4.1.7`.

### 4. Assinatura e Deploy Seguro
- **O processo:** O artefacto gerado pelo build (`artifact.tar`) passa por uma etapa de assinatura digital utilizando o **Cosign**. 
- **Verificação:** Antes de prosseguir para a publicação, a esteira verifica a validade da assinatura do artefacto para garantir que ele não foi adulterado, realizando em seguida o deploy automático para o GitHub Pages.

---

## URL de Produção
> [Site publicado no GitHub Pages](https://thegaan.github.io/projeto-devsecop-desafio/) *