# 📘 Documentação — Syscalls no Linux (x86-64)

## 1. O que é um syscall

**Syscall (System Call)** é o mecanismo que permite que um programa em **modo usuário (user space)** solicite serviços ao **kernel (kernel space)**.

### Exemplos de serviços do kernel
- Acesso a arquivos (`open`, `read`, `write`)
- Criação de processos (`fork`, `execve`)
- Comunicação (`socket`, `pipe`)
- Controle de memória (`mmap`, `brk`)
- Encerramento de programas (`exit`)

> ❗ Programas **não acessam hardware diretamente**. Todo acesso é mediado pelo kernel via syscalls.

---

## 2. User Space × Kernel Space

| Espaço | Características |
|------|----------------|
| **User Space** | Aplicações comuns, sem acesso direto ao hardware |
| **Kernel Space** | Kernel, drivers, gerenciamento de recursos |

A transição entre esses dois espaços ocorre **apenas por mecanismos controlados**, como syscalls.

---

## 3. Como um syscall funciona (visão geral)

1. O programa coloca os argumentos nos registradores
2. Coloca o **número do syscall** em `RAX`
3. Executa a instrução `syscall`
4. O kernel:
   - valida o pedido
   - executa a função correspondente
   - retorna o resultado
5. O retorno vem em `RAX`

---

## 4. Número do syscall (x86-64 Linux)

Cada syscall possui um **número único**.

### Exemplos comuns

| Syscall | Número |
|------|------|
| `read` | 0 |
| `write` | 1 |
| `open` | 2 |
| `close` | 3 |
| `stat` | 4 |
| `mmap` | 9 |
| `exit` | 60 |
| `fork` | 57 |
| `execve` | 59 |
| `openat` | 257 |

📌 Lista oficial no sistema: /usr/include/asm/unistd_64.h


---

## 5. Convenção de chamadas de syscall (x86-64)

### Registradores utilizados

| Ordem | Registrador |
|----|-----------|
| Número do syscall | `RAX` |
| 1º argumento | `RDI` |
| 2º argumento | `RSI` |
| 3º argumento | `RDX` |
| 4º argumento | `R10` |
| 5º argumento | `R8` |
| 6º argumento | `R9` |

### Execução do syscall

```asm
syscall
```
### Retorno

- Sucesso: valor retornado em RAX
- Erro: valor negativo (-errno)


<br>

---

## 6. Instrução syscall

A instrução syscall:
- Faz a troca de modo usuário → modo kernel
- Salva o estado da CPU
- Executa o handler do kernel
- Retorna automaticamente ao user space

### Registradores afetados

- RCX e R11 são sempre sobrescritos
- Devem ser preservados se forem usados

---

## 7. Flags de syscalls (exemplo: open)

As flags controlam o comportamento do syscall.

Flags comuns do `open`
| Flag       | Valor   | Descrição         |
| ---------- | ------- | ----------------- |
| `O_RDONLY` | `0x0`   | Somente leitura   |
| `O_WRONLY` | `0x1`   | Somente escrita   |
| `O_RDWR`   | `0x2`   | Leitura e escrita |
| `O_CREAT`  | `0x40`  | Cria arquivo      |
| `O_TRUNC`  | `0x200` | Zera o arquivo    |
| `O_APPEND` | `0x400` | Escrita no final  |


As flags podem ser combinadas com OR (|).

---

## 8. Exemplo prático de syscall
```asm
mov     rax, 1        # write
mov     rdi, 1        # stdout
lea     rsi, [msg]
mov     rdx, 13
syscall
```

---

## 9. Tratamento de erros

Se `RAX < 0`, ocorreu erro.
```asm
test rax, rax
js erro
```

O valor retornado é `-errno`, por exemplo:

| Código | Significado                 |
| ------ | --------------------------- |
| `-2`   | ENOENT (arquivo não existe) |
| `-13`  | EACCES (permissão negada)   |

---

## 10. open vs openat
`open` (legado)
```asm
mov rax, 2
```
`openat` (moderno e recomendado)
```asm
mov rax, 257
mov rdi, -100     # AT_FDCWD
```
Internamente, o kernel moderno utiliza openat.

---

## 11. Syscalls mais usados em Assembly 
| Categoria | Syscalls                         |
| --------- | -------------------------------- |
| Arquivos  | `open`, `read`, `write`, `close` |
| Processos | `fork`, `execve`, `exit`         |
| Memória   | `mmap`, `brk`                    |
| Tempo     | `nanosleep`, `clock_gettime`     |
| IPC       | `pipe`, `socket`                 |

---

## 12. Diferença: syscall × libc
| libc (`printf`) | syscall          |
| --------------- | ---------------- |
| Alto nível      | Baixo nível      |
| Usa buffer      | Sem buffer       |
| Portável        | Específico do SO |
| Mais lento      | Mais rápido      |

Exemplo:
```asm
printf → write (syscall)
```

---

## 13. Boas práticas
- Sempre verificar erros
- Preservar registradores quando necessário
- Preferir openat
- Usar buffers adequados
- Entender bem flags e permissões

---

## 14. Onde estudar mais
- man 2 syscall
- man 2 open
- Linux Kernel Documentation
- Linux System Programming — Robert Love
- Understanding the Linux Kernel — Bovet & Cesati

---

## 15. Conclusão

Syscalls são:
- A ponte entre aplicações e o kernel
- Fundamentais para entender sistemas operacionais
- Essenciais para programação em Assembly