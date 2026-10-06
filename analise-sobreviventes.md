# Análise dos Mutantes Sobreviventes

---

## 1. `divisao` (linha 8)

Teste atual: `expect(divisao(10, 2)).toBe(5); expect(() => divisao(5, 0)).toThrow();`

### Mutante 1: StringLiteral

```diff
- if (b === 0) throw new Error('Divisão por zero não é permitida.');
+ if (b === 0) throw new Error("");
```

1. **O que mudou:** a mensagem do erro virou uma string vazia.
2. **Por que não falhou:** `toThrow()` sem argumento só confere que **algum** erro foi lançado. Não olha a mensagem.
3. **Teste que mata:**
  ```js
   expect(() => divisao(5, 0)).toThrow('Divisão por zero não é permitida.');
  ```

---

## 2. `raizQuadrada` (linha 13)

Teste atual: `expect(raizQuadrada(16)).toBe(4);`

### Mutante 2: EqualityOperator

```diff
- if (n < 0) throw new Error(...);
+ if (n <= 0) throw new Error(...);
```

1. **O que mudou:** o zero passou a ser tratado como inválido.
2. **Por que não falhou:** o teste só usa `16`. Com `16`, `16 < 0` e `16 <= 0` são ambos `false`. A diferença só aparece na fronteira, `n = 0`.
3. **Teste que mata:**
  ```js
   expect(raizQuadrada(0)).toBe(0); // mutante lança erro
  ```

### Mutante 3: ConditionalExpression

```diff
- if (n < 0) throw new Error(...);
+ if (false) throw new Error(...);
```

1. **O que mudou:** a validação de número negativo foi removida.
2. **Por que não falhou:** nenhum teste passa um número negativo, então o `throw` nunca é necessário.
3. **Teste que mata:**
  ```js
   expect(() => raizQuadrada(-4)).toThrow(); // mutante retorna NaN
  ```

---

## 3. `fatorial` (linhas 18 e 19)

Teste atual: `expect(fatorial(4)).toBe(24);`

### Mutante 4: EqualityOperator (linha 18)

```diff
- if (n < 0) throw new Error(...);
+ if (n <= 0) throw new Error(...);
```

1. **O que mudou:** `fatorial(0)` passou a lançar erro.
2. **Por que não falhou:** o teste só usa `4`, longe da fronteira `0`.
3. **Teste que mata:**
  ```js
   expect(fatorial(0)).toBe(1); // mutante lança erro
  ```

### Mutante 5: ConditionalExpression (linha 18)

```diff
- if (n < 0) throw new Error(...);
+ if (false) throw new Error(...);
```

1. **O que mudou:** a validação de número negativo foi removida.
2. **Por que não falhou:** nenhum teste usa número negativo.
3. **Teste que mata:**
  ```js
   expect(() => fatorial(-1)).toThrow(); // mutante retorna 1
  ```

### Mutantes 6 a 9 ⚖️ (equivalentes, linha 19)

```diff
- if (n === 0 || n === 1) return 1;
+ if (false) return 1;              // mutante 6
+ if (n === 0 && n === 1) return 1; // mutante 7
+ if (false || n === 1) return 1;   // mutante 8
+ if (n === 0 || false) return 1;   // mutante 9
```

1. **O que mudou:** o retorno antecipado para `0` e `1` foi desativado, no todo ou em parte.
2. **Por que não falhou:** o retorno antecipado é só um atalho. Sem ele, para `n = 0` ou `n = 1` o laço
 `for (let i = 2; i <= n; i++)` não executa nenhuma vez, e a função retorna `resultado = 1`, o mesmo valor.
 **São mutantes equivalentes.**
3. **Teste que mata:** nenhum, já que o comportamento é idêntico para toda entrada. Mesmo assim vale incluir
 `expect(fatorial(0)).toBe(1)` e `expect(fatorial(1)).toBe(1)` para documentar os casos base. A forma de
 eliminar esses mutantes seria remover a linha 19, que é redundante.

---

## 4. `mediaArray` (linha 25)

Teste atual: `expect(mediaArray([10, 20, 30])).toBe(20);`

### Mutante 10: ConditionalExpression

```diff
- if (numeros.length === 0) return 0;
+ if (false) return 0;
```

1. **O que mudou:** o tratamento de array vazio foi removido.
2. **Por que não falhou:** o teste só usa um array com elementos.
3. **Teste que mata:**
  ```js
   expect(mediaArray([])).toBe(0); // mutante retorna 0/0 = NaN
  ```

---

## 5. `maximoArray` e `minimoArray` (linhas 34 e 38)

Testes atuais: `expect(maximoArray([1, 50, 10])).toBe(50);` e `expect(minimoArray([10, 2, 100])).toBe(2);`

### Mutante 11: ConditionalExpression (`maximoArray`)

```diff
- if (numeros.length === 0) throw new Error('Array vazio не possui valor máximo.');
+ if (false) throw new Error(...);
```

1. **O que mudou:** a validação de array vazio foi removida.
2. **Por que não falhou:** nenhum teste passa array vazio.
3. **Teste que mata:**
  ```js
   expect(() => maximoArray([])).toThrow(); // mutante retorna Math.max() = -Infinity
  ```

### Mutante 12: ConditionalExpression (`minimoArray`)

```diff
- if (numeros.length === 0) throw new Error('Array vazio не possui valor mínimo.');
+ if (false) throw new Error(...);
```

1. **O que mudou:** a validação de array vazio foi removida.
2. **Por que não falhou:** nenhum teste passa array vazio.
3. **Teste que mata:**
  ```js
   expect(() => minimoArray([])).toThrow(); // mutante retorna Math.min() = Infinity
  ```

---

## 6. `isPar` e `isImpar` (linhas 43 e 44)

Testes atuais: `expect(isPar(100)).toBe(true);` e `expect(isImpar(7)).toBe(true);`

### Mutante 13: ConditionalExpression (`isPar`)

```diff
- function isPar(n) { return n % 2 === 0; }
+ function isPar(n) { return true; }
```

1. **O que mudou:** a função passou a dizer que todo número é par.
2. **Por que não falhou:** o teste só confere um caso que deve dar `true`. O mutante sempre dá `true`.
3. **Teste que mata:**
  ```js
   expect(isPar(7)).toBe(false);
  ```

### Mutante 14: ConditionalExpression (`isImpar`)

```diff
- function isImpar(n) { return n % 2 !== 0; }
+ function isImpar(n) { return true; }
```

1. **O que mudou:** a função passou a dizer que todo número é ímpar.
2. **Por que não falhou:** o teste só confere um caso que deve dar `true`.
3. **Teste que mata:**
  ```js
   expect(isImpar(4)).toBe(false);
  ```

### Mutante 15: ArithmeticOperator (`isImpar`)

```diff
- return n % 2 !== 0;
+ return n * 2 !== 0;
```

1. **O que mudou:** o resto da divisão virou multiplicação. O resultado é `true` para qualquer número diferente de zero.
2. **Por que não falhou:** com `7`, `7 * 2 = 14 !== 0` dá `true`, o mesmo resultado esperado.
3. **Teste que mata:** o mesmo do mutante 14:
  ```js
   expect(isImpar(4)).toBe(false); // mutante: 8 !== 0 → true
  ```

---

## 7. `isPrimo` (linhas 73 a 75)

Teste atual: `expect(isPrimo(7)).toBe(true);`

Todos os mutantes desta função sobrevivem pelo mesmo motivo: o teste só usa um número **primo**, que deve dar `true`,
e a última linha da função já é `return true`. Qualquer mutação que impeça a função de retornar `false` passa despercebida.

### Mutante 16: ConditionalExpression (linha 73)

```diff
- if (n <= 1) return false;
+ if (false) return false;
```

1. **O que mudou:** a regra de que números menores ou iguais a 1 não são primos foi removida.
2. **Por que não falhou:** o teste não usa `0`, `1` nem negativos.
3. **Teste que mata:**
  ```js
   expect(isPrimo(1)).toBe(false); // mutante: laço não roda → true
  ```

### Mutante 17: EqualityOperator (linha 73)

```diff
- if (n <= 1) return false;
+ if (n < 1) return false;
```

1. **O que mudou:** o `1` passou a ser considerado primo.
2. **Por que não falhou:** o teste não usa a fronteira `n = 1`.
3. **Teste que mata:**
  ```js
   expect(isPrimo(1)).toBe(false);
  ```

### Mutante 18: ConditionalExpression (linha 74)

```diff
- for (let i = 2; i < n; i++) {
+ for (let i = 2; false; i++) {
```

1. **O que mudou:** o laço que procura divisores nunca executa.
2. **Por que não falhou:** com `7`, o laço não encontra divisor e a função chega ao `return true` do mesmo jeito.
3. **Teste que mata:**
  ```js
   expect(isPrimo(9)).toBe(false); // mutante: não testa divisores → true
  ```

### Mutante 19: EqualityOperator (linha 74)

```diff
- for (let i = 2; i < n; i++) {
+ for (let i = 2; i >= n; i++) {
```

1. **O que mudou:** a condição do laço foi invertida. Para `n > 2`, o laço não executa.
2. **Por que não falhou:** mesmo motivo do mutante 18.
3. **Teste que mata:**
  ```js
   expect(isPrimo(9)).toBe(false);
  ```

### Mutante 20: BlockStatement (linha 74)

```diff
- for (let i = 2; i < n; i++) {
-   if (n % i === 0) return false;
- }
+ for (let i = 2; i < n; i++) {}
```

1. **O que mudou:** o corpo do laço foi apagado. O laço roda, mas não verifica nada.
2. **Por que não falhou:** mesmo motivo do mutante 18.
3. **Teste que mata:**
  ```js
   expect(isPrimo(9)).toBe(false);
  ```

### Mutante 21: ArithmeticOperator (linha 75)

```diff
- if (n % i === 0) return false;
+ if (n * i === 0) return false;
```

1. **O que mudou:** o teste de divisibilidade virou uma multiplicação, que nunca dá zero para `n` e `i` positivos.
2. **Por que não falhou:** com `7`, a condição original também nunca é verdadeira.
3. **Teste que mata:**
  ```js
   expect(isPrimo(9)).toBe(false); // mutante: 9 * i nunca é 0 → true
  ```

### Mutante 22: ConditionalExpression (linha 75)

```diff
- if (n % i === 0) return false;
+ if (false) return false;
```

1. **O que mudou:** a função nunca encontra divisor.
2. **Por que não falhou:** mesmo motivo do mutante 18.
3. **Teste que mata:**
  ```js
   expect(isPrimo(9)).toBe(false);
  ```

---

## 8. `produtoArray` (linha 84)

Teste atual: `expect(produtoArray([2, 3, 4])).toBe(24);`

### Mutante 23 ⚖️ (equivalente): ConditionalExpression

```diff
- if (numeros.length === 0) return 1;
+ if (false) return 1;
```

1. **O que mudou:** o tratamento de array vazio foi removido.
2. **Por que não falhou:** `reduce((acc, val) => acc * val, 1)` com valor inicial `1` já retorna `1` para um array vazio.
 O `if` é redundante. **É um mutante equivalente.**
3. **Teste que mata:** nenhum. Ainda assim vale documentar o caso com `expect(produtoArray([])).toBe(1)`.

---

## 9. `clamp` (linhas 88 e 89)

Teste atual: `expect(clamp(5, 0, 10)).toBe(5);`

O teste só usa um valor **dentro** do intervalo. Nenhum dos dois `if` decide o resultado nesse caso.

### Mutante 24: ConditionalExpression (linha 88)

```diff
- if (valor < min) return min;
+ if (false) return min;
```

1. **O que mudou:** o limite inferior deixou de ser aplicado.
2. **Por que não falhou:** o teste nunca usa um valor abaixo do mínimo.
3. **Teste que mata:**
  ```js
   expect(clamp(-5, 0, 10)).toBe(0); // mutante retorna -5
  ```

### Mutante 25 ⚖️ (equivalente): EqualityOperator (linha 88)

```diff
- if (valor < min) return min;
+ if (valor <= min) return min;
```

1. **O que mudou:** quando `valor === min`, o mutante retorna `min` em vez de seguir para `return valor`.
2. **Por que não falhou:** se `valor === min`, retornar `min` ou `valor` é a mesma coisa. **É um mutante equivalente.**
3. **Teste que mata:** nenhum. O caso `clamp(0, 0, 10)` dá `0` nas duas versões.

### Mutante 26: ConditionalExpression (linha 89)

```diff
- if (valor > max) return max;
+ if (false) return max;
```

1. **O que mudou:** o limite superior deixou de ser aplicado.
2. **Por que não falhou:** o teste nunca usa um valor acima do máximo.
3. **Teste que mata:**
  ```js
   expect(clamp(15, 0, 10)).toBe(10); // mutante retorna 15
  ```

### Mutante 27 ⚖️ (equivalente): EqualityOperator (linha 89)

```diff
- if (valor > max) return max;
+ if (valor >= max) return max;
```

1. **O que mudou:** quando `valor === max`, o mutante retorna `max` em vez de `valor`.
2. **Por que não falhou:** os dois valores são iguais nesse caso. **É um mutante equivalente.**
3. **Teste que mata:** nenhum.

---

## 10. `isDivisivel` (linha 92)

Teste atual: `expect(isDivisivel(10, 2)).toBe(true);`

### Mutante 28: ConditionalExpression

```diff
- return dividendo % divisor === 0;
+ return true;
```

1. **O que mudou:** a função passou a dizer que todo número é divisível por qualquer outro.
2. **Por que não falhou:** o teste só confere um caso que deve dar `true`.
3. **Teste que mata:**
  ```js
   expect(isDivisivel(10, 3)).toBe(false);
  ```

---

## 11. `celsiusParaFahrenheit` (linha 93)

Teste atual: `expect(celsiusParaFahrenheit(0)).toBe(32);`

### Mutante 29: ArithmeticOperator

```diff
- return (celsius * 9/5) + 32;
+ return (celsius * 9 * 5) + 32;
```

### Mutante 30: ArithmeticOperator

```diff
- return (celsius * 9/5) + 32;
+ return (celsius / 9/5) + 32;
```

1. **O que mudou:** o fator de conversão `9/5` foi trocado por `9 * 5` (mutante 29) ou `/ 9 / 5` (mutante 30).
2. **Por que não falhou:** o teste usa `0`. Zero multiplicado ou dividido por qualquer coisa continua zero,
 então as três versões dão `0 + 32 = 32`. A entrada **anula** justamente a parte que foi mutada.
3. **Teste que mata (os dois):**
  ```js
   expect(celsiusParaFahrenheit(100)).toBe(212); // mutantes: 4532 e ≈34.2
  ```

---

## 12. `fahrenheitParaCelsius` (linha 94)

Teste atual: `expect(fahrenheitParaCelsius(32)).toBe(0);`

### Mutante 31: ArithmeticOperator

```diff
- return (fahrenheit - 32) * 5/9;
+ return (fahrenheit - 32) * 5 * 9;
```

### Mutante 32: ArithmeticOperator

```diff
- return (fahrenheit - 32) * 5/9;
+ return (fahrenheit - 32) / 5/9;
```

1. **O que mudou:** o fator `5/9` foi trocado por `5 * 9` (mutante 31) ou `/ 5 / 9` (mutante 32).
2. **Por que não falhou:** com `32`, `(32 - 32) = 0`, e zero vezes qualquer fator dá zero. Mesmo problema da conversão anterior.
3. **Teste que mata (os dois):**
  ```js
   expect(fahrenheitParaCelsius(212)).toBe(100); // mutantes: 8100 e 4
  ```

---

## 13. `inverso` (linha 96)

Teste atual: `expect(inverso(4)).toBe(0.25);`

### Mutante 33: ConditionalExpression

```diff
- if (n === 0) throw new Error('Não é possível inverter o número zero.');
+ if (false) throw new Error(...);
```

1. **O que mudou:** a validação do zero foi removida.
2. **Por que não falhou:** nenhum teste usa `0`.
3. **Teste que mata:**
  ```js
   expect(() => inverso(0)).toThrow(); // mutante retorna 1/0 = Infinity
  ```

---

## 14. `isMaiorQue` (linha 104)

Teste atual: `expect(isMaiorQue(10, 5)).toBe(true);`

### Mutante 34: ConditionalExpression

```diff
- return a > b;
+ return true;
```

1. **O que mudou:** a função sempre responde que `a` é maior.
2. **Por que não falhou:** o teste só confere um caso `true`.
3. **Teste que mata:**
  ```js
   expect(isMaiorQue(5, 10)).toBe(false);
  ```

### Mutante 35: EqualityOperator

```diff
- return a > b;
+ return a >= b;
```

1. **O que mudou:** valores iguais passaram a ser considerados "maior".
2. **Por que não falhou:** o teste não usa a fronteira `a === b`.
3. **Teste que mata:**
  ```js
   expect(isMaiorQue(5, 5)).toBe(false);
  ```

---

## 15. `isMenorQue` (linha 105)

Teste atual: `expect(isMenorQue(5, 10)).toBe(true);`

### Mutante 36: ConditionalExpression

```diff
- return a < b;
+ return true;
```

1. **O que mudou:** a função sempre responde que `a` é menor.
2. **Por que não falhou:** o teste só confere um caso `true`.
3. **Teste que mata:**
  ```js
   expect(isMenorQue(10, 5)).toBe(false);
  ```

### Mutante 37: EqualityOperator

```diff
- return a < b;
+ return a <= b;
```

1. **O que mudou:** valores iguais passaram a ser considerados "menor".
2. **Por que não falhou:** o teste não usa a fronteira `a === b`.
3. **Teste que mata:**
  ```js
   expect(isMenorQue(5, 5)).toBe(false);
  ```

---

## 16. `isEqual` (linha 106)

Teste atual: `expect(isEqual(7, 7)).toBe(true);`

### Mutante 38: ConditionalExpression

```diff
- return a === b;
+ return true;
```

1. **O que mudou:** a função sempre responde que os valores são iguais.
2. **Por que não falhou:** o teste só confere um caso `true`.
3. **Teste que mata:**
  ```js
   expect(isEqual(7, 8)).toBe(false);
  ```

---

## 17. `medianaArray` (linhas 108, 109 e 111)

Teste atual: `expect(medianaArray([1, 2, 3, 4, 5])).toBe(3);`

O array do teste já está **ordenado** e tem tamanho **ímpar**. Por isso a ordenação não faz diferença,
e o ramo de tamanho par nunca é usado.

### Mutante 39: ConditionalExpression (linha 108)

```diff
- if (numeros.length === 0) throw new Error('Array vazio не possui mediana.');
+ if (false) throw new Error(...);
```

1. **O que mudou:** a validação de array vazio foi removida.
2. **Por que não falhou:** nenhum teste passa array vazio.
3. **Teste que mata:**
  ```js
   expect(() => medianaArray([])).toThrow(); // mutante retorna NaN
  ```

### Mutante 40: MethodExpression (linha 109)

```diff
- const sorted = [...numeros].sort((a, b) => a - b);
+ const sorted = [...numeros];
```

### Mutante 41: ArrowFunction (linha 109)

```diff
- const sorted = [...numeros].sort((a, b) => a - b);
+ const sorted = [...numeros].sort(() => undefined);
```

### Mutante 42: ArithmeticOperator (linha 109)

```diff
- const sorted = [...numeros].sort((a, b) => a - b);
+ const sorted = [...numeros].sort((a, b) => a + b);
```

1. **O que mudou:** a ordenação foi removida (40) ou trocada por um comparador que não ordena (41 e 42).
2. **Por que não falhou:** o array do teste, `[1, 2, 3, 4, 5]`, **já está ordenado**. Ordenar ou não dá o mesmo resultado.
3. **Teste que mata (os três):**
  ```js
   expect(medianaArray([5, 1, 3])).toBe(3); // mutantes mantêm [5, 1, 3] → retornam 1
  ```

   *(Comportamento confirmado no Node: com os comparadores mutados, `[5, 1, 3]` continua fora de ordem.)*

### Mutante 43: ConditionalExpression (linha 111)

```diff
- if (sorted.length % 2 === 0) {
+ if (false) {
```

### Mutante 44: ArithmeticOperator (linha 111)

```diff
- if (sorted.length % 2 === 0) {
+ if (sorted.length * 2 === 0) {
```

1. **O que mudou:** o ramo que calcula a mediana de arrays de tamanho par foi desativado. No mutante 44,
 `length * 2 === 0` só é verdadeiro para um array vazio, que nunca chega a essa linha.
2. **Por que não falhou:** o teste usa um array de tamanho **ímpar**. A condição original já dava `false`.
3. **Teste que mata (os dois):**
  ```js
   expect(medianaArray([1, 2, 3, 4])).toBe(2.5); // mutantes retornam sorted[2] = 3
  ```

