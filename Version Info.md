# Wordflow — vocabulário com IA local

## Iniciar no Linux

É necessário ter Python 3.9 ou posterior, Ollama e o modelo Qwen3:4b.
Não é necessário instalar pacotes Python ou usar pip.

1. Extraia o ZIP inteiro. Mantenha `server.py`, `config.json` e `index.html` juntos.
2. Abra um terminal dentro da pasta `wordflow` extraída.
3. Se ainda precisar baixar o modelo: `ollama pull qwen3:4b`.
4. Inicie a aplicação:

```bash
python3 server.py
```

5. Abra **http://localhost:8000** no navegador.
6. Escolha o nível e o tema; clique em **Nova palavra**.

Não abra o HTML com duplo clique: a aplicação depende do servidor Python.
Deixe o terminal aberto. Use Ctrl+C para encerrar.
Se o Ollama não estiver ativo, execute `ollama serve` em outro terminal.
Se ele disser que a porta 11434 já está em uso, provavelmente já está rodando.

## O que esta versão faz

- Usa o Ollama local com `think: false` e JSON Schema.
- Solicita uma palavra de acordo com o tema e o nível declarados; a escolha da IA não é um sorteio estatisticamente uniforme.
- Exibe palavra, tradução, frase em inglês e tradução da frase.
- Rejeita campos ausentes, frases copiadas, palavra ausente na frase, nível diferente e duplicatas.
- Tenta novamente uma vez caso a resposta seja inválida.
- Salva o histórico no SQLite, arquivo `wordflow.db`, criado automaticamente junto ao servidor.
- Exclui das novas gerações todas as palavras já apresentadas, inclusive as marcadas como conhecidas.
- Guarda até oito cartões recentes para exibição; o banco preserva todos para evitar duplicatas.
- Mantém as preferências da interface no navegador. A meta conta gerações bem-sucedidas no dia local do servidor, não palavras aprendidas.

As verificações são estruturais e heurísticas: não comprovam idioma, correção da tradução ou nível pedagógico. Confira o conteúdo antes de estudar. O botão de palavra conhecida registra um feedback; ainda não há adaptação automática de nível nem integração com revisões do Anki.

O banco é de um único usuário. Todos os navegadores que acessarem a mesma instância compartilham seu histórico. O histórico antigo da demonstração HTML não é importado, pois usava apenas exemplos locais.

## Configuração

Edite `config.json` com o servidor parado e inicie novamente:

```json
{
  "ollama_url": "http://127.0.0.1:11434",
  "model": "qwen3:4b",
  "port": 8000,
  "timeout_seconds": 120
}
```

- `ollama_url`: endereço base do Ollama, sem `/api/chat` no final.
- `model`: nome exato do modelo instalado.
- `port`: porta da interface Python.
- `timeout_seconds`: tempo máximo por tentativa. Até duas tentativas podem ocorrer.

Também é possível sobrescrever endereço e modelo por variáveis de ambiente:

```bash
OLLAMA_URL=http://127.0.0.1:11434 OLLAMA_MODEL=qwen3:4b python3 server.py
```

Para outra porta, use `WORDFLOW_PORT=8001 python3 server.py`.
O modelo fica carregado por até 10 minutos de inatividade, conforme suporte do Ollama; isso pode acelerar chamadas seguintes e ocupa memória durante esse período.

## Futuro servidor Termux

Nesta etapa o servidor Python aceita apenas conexões locais no computador. A lógica de chamada da IA está isolada na função `ask_ollama`, e o endereço e o modelo são configuráveis.

Se futuramente houver um Ollama compatível acessível no celular, será possível configurar seu endereço. Se usar llama.cpp no Termux, será necessário adaptar `ask_ollama` para a API do llama-server; trocar apenas a URL não basta. Também precisaremos configurar o acesso pela rede local e testar o modelo e a memória disponível no celular de 4 GB. O Qwen3:4b escolhido para o computador não é uma promessa de viabilidade no celular.

## Problemas comuns

- **Não conecta ao Ollama:** confira o endereço e se `ollama serve` está ativo.
- **Modelo não encontrado:** execute `ollama pull qwen3:4b`, ou o modelo configurado.
- **IA demorou demais:** a primeira carga pode ser lenta; aumente o tempo em `config.json` se necessário.
- **Cartão inválido em duas tentativas:** nenhum cartão é salvo; tente novamente ou compare outro modelo.
- **Porta 8000 ocupada:** escolha outra porta.
- **Palavras repetidas sugeridas pela IA:** o banco rejeita duplicatas; depois de duas falhas a interface informa o erro. A lista enviada no prompt é limitada para não sobrecarregar o contexto, mas a verificação usa todo o histórico.

## Anki

O campo de baralho está reservado para a próxima etapa. **Nenhum cartão é enviado ao Anki nesta versão.**

## Validação desta entrega

Testes automatizados com um Ollama simulado verificam o protocolo de integração, persistência, duplicatas, repetição da tentativa, erros HTTP e entradas inválidas. O modelo real instalado no computador do usuário não está acessível no ambiente de desenvolvimento; a qualidade e o desempenho reais devem ser conferidos no primeiro uso local.
