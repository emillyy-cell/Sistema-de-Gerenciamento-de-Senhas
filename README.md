# Sistema de Gerenciamento de Senhas de Atendimento Clínico

Projeto em linguagem C (Desafio – Algoritmos e Pensamento Computacional, UDF).
Cadastra pacientes, classifica por prioridade, emite senhas e controla a ordem de atendimento.

## Como compilar e executar

```bash
gcc -Wall -Wextra -o senhas main.c
./senhas
# ou, com o exemplo pronto:
./senhas < exemplo_entrada.txt
```

## Regras de classificação

| Código | Categoria | Quem entra |
|---|---|---|
| `U001` | Urgente | Idade **superior a 80** anos (80 exatos não é urgente) |
| `P001` | Preferencial | Gestante, lactante, necessidade especial, obesidade, mulher com 60+ anos, homem com 65+ anos |
| `N001` | Normal | Nenhuma condição de prioridade |

- O teste de **Urgente vem primeiro**: uma mulher de 85 anos também seria preferencial, mas a categoria mais prioritária vence.
- A numeração da senha é sequencial **por categoria** (N001, N002, ...).
- Perguntas sobre gestante e lactante só são feitas quando o gênero é F.

## Regras de chamada

1. Urgentes antes de Preferenciais, e Preferenciais antes de Normais.
2. Dentro de cada categoria vale a ordem de emissão (**FIFO**, fila circular).
3. Senha já atendida não volta para a fila.
4. Só é possível chamar outra senha depois de finalizar a atual.
5. Aceita vários cadastros durante a execução (até 100 por fila).

> A classificação por idade é só uma regra de programação desta prova e não substitui avaliação clínica.

## Estrutura do código (modularização)

| Função | Responsabilidade |
|---|---|
| `cadastrarPaciente()` | Lê os dados, classifica, gera a senha e enfileira |
| `classificar()` | Retorna `'U'`, `'P'` ou `'N'` |
| `gerarSenha()` | Monta a senha sequencial (ex.: `P002`) |
| `enfileirar()` / `desenfileirar()` | Operações da fila FIFO |
| `proximaFila()` | Decide qual fila é chamada primeiro |
| `consultarProxima()`, `chamarPaciente()`, `mostrarEmAtendimento()`, `finalizarAtendimento()` | Registro de atendimento |
| `exibirPainel()` | Painel textual |

Estruturas: `Paciente` (struct) e `Fila` (struct com vetor + índices início/fim/quantidade).

## Menu

```
1 - Cadastrar paciente / emitir senha
2 - Consultar proxima senha
3 - Chamar paciente
4 - Senha em atendimento
5 - Finalizar atendimento
6 - Painel de chamada
0 - Sair
```

## Exemplo demonstrativo

Cadastros (nesta ordem): homem 30 anos → `N001`; mulher 28 gestante → `P001`; homem 82 → `U001`; mulher 62 → `P002`; homem 40 obeso → `P003`.

Painel logo após os cadastros:

```
Proximas      : U001 P001 P002 P003 N001
Em espera     : Urgente=1 | Preferencial=3 | Normal=1
```

Ordem de chamada resultante: **U001 → P001 → P002 → P003 → N001**.

Após chamar e finalizar `U001` e chamar `P001`:

```
Senha chamada : P001
Proximas      : P002 P003 N001
Em espera     : Urgente=0 | Preferencial=2 | Normal=1
```

A saída completa está em `exemplo_saida.txt`.
