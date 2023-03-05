# PCS3616 - Laboratório 7 - ASM 1

Na aula de hoje começamos o estudo e uso da linguagem de montagem da
MVN, o ASM (assembly), e da função de pilha implementada na MVN.
A partir desta aula você só escreverá códigos em ASM, e para isso
precisamos usar os módulos Montador, Ligador e Relocador.

## Instalação das ferramentas

Diferentemente do monitor, essas ferramentas foram escritas em uma
linguagem compilada ([Rust](https://www.rust-lang.org/)), então é
somente necessário instalar um executável. Esse executável está
disponível na aba "Releases" do
[repositório das ferramentas](https://github.com/PCS3616/mvn-mounter).

É recomendado que você realize download do executável em algum diretório
no seu `$PATH`, ou que coloque ele dentro do diretório `submission` do
seu repositório. Em ambos os casos, é necessário indicar ao sistema
operacional que o arquivo é executável fazendo
```shell
$ chmod +x <EXECUTAVEL>
```

Caso decida colocar o executável dentro de `submission`, você deverá usar
uma referência relativa ao caminho dele no momento da execução; se o
executável se chamar `mvn-cli` por exemplo, você deve executar, de dentro
do diretório `submission`
```shell
$ ./mvn-cli <ARGS...>
```

## Instruções de uso das ferramentas

Vale mencionar que a interface das ferramentas possui opções `help` e
`--help` que nomeiam os argumentos necessários.

Você não precisa se preocupar com os arquivos INT, LIG ou MVN gerados no
diretório `submission`, já que eles devem ser ignorados automaticamente
pelo Git e não serão avaliados.

### Programas exclusivamente com endereços absolutos

Para executar programas escritos em linguagem de montagem, eles precisam
estar em linguagem de máquina. Essa transposição deve ser feita usando o
montador como no exemplo a seguir:
```shell
$ mvn-cli assemble -i absoluto.asm > absoluto.mvn
```

### Programas com endereços relocáveis

O processo é mais complexo com endereços relocáveis.
Para demonstrar como usar as ferramentas, vamos assumir que um programa
foi desenvolvido com os módulos `principal.asm` e `secundario.asm`.

1. Em primeiro lugar, é necessário gerar arquivos INT a partir do montador:
  ```shell
  $ mvn-cli assemble -i principal.asm > principal.int
  $ mvn-cli assemble -i secundario.asm > secundario.int
  ```

2. Em seguida, é necessário ligar os arquivos usando o ligador para gerar um
   arquivo LIG caso todos os símbolos estejam resolvidos.
   No lugar da flag `--complete`, é possível passar a flag `--partial` para
   realizar ligação parcial, usada para gerar bibliotecas e não executáveis
  ```shell
  $ mvn-cli link -i principal.int -i secundario.int --complete > programa.lig
  ```

3. Por fim, é necessário relocar o programa LIG ligado para gerar um
   executável MVN com endereços absolutos.
   É obrigatório passar a base de relocação (`--base` ou `-b`), ainda que em
   geral utilizemos 0.
  ```shell
  $ mvn-cli assemble -i programa.lig --base 0 > programa.mvn
  ```

### Execução do código gerado

Executar o código MVN gerado deve ser feito da mesma forma que com o
código MVN escrito manualmente.

## Exercícios

1.  No laboratório 5, você escreveu as sub-rotinas OP2MNEM e MNEM2OP.
    Neste primeiro exercício, você deverá escrever uma versão modificada
    deste programa, desta vez usando a linguagem de montagem da MVN.

    Especificações do programa:

    -   Nome do arquivo: **op-mnem.asm**

    -   Layout da memória:

    | **Endereço/Rótulo** | **Conteúdo**                          |
    |---------------------|---------------------------------------|
    | `0x000`             | Jump para o programa principal        |
    | OPCODE              | Variável OPCODE (variável \"global\") |
    | MNEM                | Variável MNEM (variável \"global\")   |
    | OP2MNEM             | Sub-rotina OP2MNEM                    |
    | MNEM2OP             | Sub-rotina MNEM2OP                    |
    | `0x0300` MAIN       | Programa principal                    |
    | TABELA              | Tabela de mnemônicos                  |

    **Entrega:** arquivo **op-mnem.asm**

2.  Agora vamos aprender a usar a pilha implementada pela MVN.

    A pilha implementada na MVN é muito simples e deve ser operada com
    cautela. Para utilizar as funções auxiliares e implementar a pilha você
    deve fazer uso da instrução OS (supervisor), o código da função a ser
    executada pelo supervisor deve ser carregado no acumulador e o código de
    operação da função a ser passado no operando de OS é `0x57`. O uso da
    instrução deve se dar da seguinte forma: `OS /<NUM_ARG><FUNC>`

    ```
    NUM_ARG = 0    (para funções de get, sem parâmetro)        ; Ex. de chamada: OS /057
            = 1    (para funções de set, com um parâmetro)     ; Ex. de chamada: OS /157
    FUNC    = 0x57                                             ; indica qual a função a ser empregada
    ```

    A pilha será implementada de forma decrescente, diferente daquela que
    vocês utilizaram na disciplina de estrutura de dados. Ou seja, a cada
    mudança do ponteiro deve-se decrementar o valor do endereço. E o valor
    do decremento é de dois (2) e não de um (1) porque na MVN os dados
    ocupam sempre dois bytes.

    O endereço de stack-pointer (SP) da pilha estará inicializado quando vocês
    forem iniciar as operações, para conhecer seu valor empregue a função
    `0x0`, para preencher o novo valor (decrementando de dois) empregue a
    função `0x1`. A pilha deverá ter seu stack-pointer sempre apontando para o
    próximo valor livre, de forma a que sempre se possa inserir algum valor
    na pilha sem necessitar alterar seu ponteiro. Assim, para se colocar um
    valor no endereço apontado pelo SP empregue a função `0x3` e
    posteriormente corrija o SP decrementando dois do valor
    atual. Para capturar um valor do topo da pilha empregue a função `0x2`,
    mas antes corrija o valor do stack-pointer incrementando dois ao valor
    atual.

    Para chamar uma das funções basta preencher o acumulador com o valor
    correspondente. As funções para uso da pilha são:

    | **Função**    | **Valor**  | **Descrição** |
    |-------------- |------------|---------------|
    | get pointer   | 0          | Salva em AC valor do SP                                                      |
    | set pointer   | 1          | Salva no SP o valor passado como argumento                                   |
    | get stack top | 2          | Salva em AC o valor no endereço de memória apontado pelo SP                  |
    | set stack top | 3          | Salva no endereço de memória apontado pelo SP o valor passado como argumento |

    *Seu trabalho* será escrever 3 rotinas:

    | Rotina        | Rótulo | Descrição |
    |---------------|--------|-----------|
    | Empilha       | `EMPI` | Lê uma variável global `VALOR` e empilha seu conteúdo              |
    | Desempilha    | `DEMP` | Desempilha um valor e salva na variável global `VALOR`             |
    | Principal     | `MAIN` | Lê uma entrada do teclado, empilha, desempilha e a escreve na tela |

    **Observações importantes:**

    - Não esqueça de exportar todos os nomes

    - Não utilize os endereços `0xF00` e `0xF02`.

    **Entrega:** arquivo **stack.asm** contendo as três rotinas.
