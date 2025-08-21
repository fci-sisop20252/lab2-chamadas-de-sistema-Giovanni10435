# Relatório do Laboratório 2 - Chamadas de Sistema

---

## Exercício 1a - Observação printf() vs 1b - write()

### Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: _1_ syscalls
- ex1b_write: _7_ syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
Há diferenças pois no printf há apenas uma chamada do sistema, que seria para printar na tela. Enquanto isso, no Write há 7 chamadas, todas para editar o arquivo.
(falta responder)
```

**3. Qual método é mais previsível? Por quê?**

```
O método mais previsível é o Write() pois ele sempre resulta em **uma** chamada do sistema (syscall) por chamada.

```

---

## Exercício 2 - Leitura de Arquivo

### Resultados da execução:
- File descriptor: _____
- Bytes lidos: _____

### Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### Análise

**1. Qual file descriptor foi usado? Por que não 0, 1 ou 2?**

```
O file descriptor utilizado foi o open() e os outros não foram utilizados pois 0, 1 e 2 são reservados para std in, out e err.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
Sabemos que o arquivo foi lido completamente quando o retorno da chamada read() é igual a 0.
```

**3. O que acontece se esquecer de fechar o arquivo?**

```
Nesse caso, o descritor ficaria aberto até executarem uma chamada para fecha-lo. Já em servidores, pode causar vazamento de descritores e travar o sistema.
```

**4. Por que verificar retorno de cada syscall?**

```
Pois as chamadas como open() read() e close() podem falhar em situações como, por exemplo, permissão negada ou arquivo inexistente. Dito isso, é mais seguro garantir o retorno do sistema a cada syscall.
```

---

## Exercício 3 - Contador com Loop

### Resultados (BUFFER_SIZE = 64):
- Linhas: _____ (esperado: 25)
- Caracteres: _____
- Chamadas read(): _____
- Tempo: _____ segundos

### Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |                 |           |
| 64          |                 |           |
| 256         |                 |           |
| 1024        |                 |           |

### Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
[Sua análise aqui]
```

**2. Como você detecta o fim do arquivo?**

```
Detectamos o fim do arquivo quando a chamada read() retorna 0.
```

**3. Todas as chamadas read() retornaram BUFFER_SIZE bytes?**

```
Não, somente as primeiras. As últimas leituras geralmente retornam menos que o BUFFER_SIZE.
```

**4. Qual é a relação entre syscalls e performance?**

```
Cada syscall é uma troca de contexto, trocando das permissões do usuário para as permissões do Kernel, ou seja, menos syscalls equivaleria a uma maior performance, já que essa troca seria feita mais efetivamente e apenas quando necessário.
```

---

## Exercício 4 - Cópia de Arquivo

### Resultados:
- Bytes copiados: _____
- Operações: _____
- Tempo: _____ segundos
- Throughput: _____ KB/s

### Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [ ] Idênticos [ ] Diferentes

### Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Pois a chamada de Write() pode gerar menos bytes do que lhe foi pedido, como por exemplo em discos cheios, logo é ideal verificar que tudo foi realmente gravado.
```

**2. Que flags são essenciais no open() do destino?**

```
O_CREAT, O_WRONLY, O_TRUNC que fazem, respectivamente: cria caso não exista, escrita e zera o arquivo caso já exista.
```

**3. O número de reads e writes é igual? Por quê?**

```
Geralmente sim, pois para cada read temos um write. A não ser que temos erro de escrita no write, como já citado no exercício 1.
```

**4. Como você saberia se o disco ficou cheio?**

```
O Write() pode retornar menos bytes que o demandado ou ele simplesmente retorna -1.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
O Kernel se fecha automaticamente caso haja o esquecimento no fechamento dos arquivos, porém se o programa for longo ou abrir muitos arquivos pode esgotar os descritores.
```

---

## Análise Geral

### Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
As syscalls sao interrupções no sistema para a alteração de permissão entre o usuário e o dispositivo/software utilizado.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Os File Descriptors funcionam como indicadores de utilização de arquivos no sistema, como abertura, diretórios e dispositivos. Eles são extremamente importantes pois são eles que possibilitam a manipulação de arquivos em termos como read(), open(), etc. utilizando apenas os números para definí-los.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
A relação entre o tamanho do buffer e performance se baseia na quantidade de syscalls que são necessárias para executar uma instrução. Diante disso, buffers maiores conseguem armazenar mais intruções, portanto podem chamar o syscall menos vezes, levando a um performance mais alto para o sistema. Pode-se dizer que são inversamente proporcionais: Quanto maior o buffer, menor a quantidade de syscalls (ou seja, melhor desempenho).
```

### Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** _cp_

**Por que você acha que foi mais rápido?**

```
Acretido que (conforme discutido no exercício 3 de análises gerais) ele tenha buffers maiores, o que resultaria em menos syscalls, logo maior desempenho.
```

---

## Entrega

Certifique-se de ter:
- [V] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [V] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
