# Atividade Compiladores

Ana Carolina Maia Atala ana.atala@unemat.br

## Funcionamento

O Analisador quebra um arquivo em linhas
para cada linha, ele roda uma subrotina que vai varrendo a string ate encontrar condicoes de parada.
A cada condição de parada, ele analisa a string ja armazenada, e começa uma nova analise,
A subrotina termina no fim da linha.

Posteriormente, um Array que guardava todas as subrotinas é exibido em uma tabela

### O analisador de condicao de parada

```js
export function lineLexer(input, currentLine) {
  if (typeof input !== 'string') {
    throw new Error('Input must be a string');
  }

  let tokens = [];
  let currentLexeme = cleanCurentLexeme(currentLine, 0);
  const maxInputLenght = input.length;

  for (const [index, character] of enumerate(input)) {
    if (END_OF_LINE.includes(character) || END_OF_WORD.includes(character)) {
      tokens.push(analyzeToken(currentLexeme, index - 1));

      currentLexeme = cleanCurentLexeme(currentLine, index + 1);
    }

    if (PUNCTUATIONS.includes(character)) {
      //verify if next character is also a /
      if (character === '/' && input[index + 1] === '/') {
        // line is a comment, ignore
        break;
      }
      //analyze previous token since a punctuation was found
      tokens.push(analyzeToken(currentLexeme, index - 1));

      //add punctiation as token
      currentLexeme = new Lexeme(character, LEXER_STATES.WORD_ITERATION, currentLine, index);
      tokens.push(analyzeToken(currentLexeme, index));

      //reset lexema
      currentLexeme = cleanCurentLexeme(currentLine, index);
      continue;
    }

    //check if we have a valid alphanumeric character
    if (!(REGEX.ALPHABETIC.test(character) || REGEX.NUMBERIC.test(character))) {
      //analyze previous token since an invalid character was found
      tokens.push(analyzeToken(currentLexeme, index - 1));

      //analyse the character as a token
      currentLexeme = new Lexeme(character, LEXER_STATES.WORD_ITERATION, currentLine, index);
      tokens.push(analyzeToken(currentLexeme, index));

      //reset lexema
      currentLexeme = cleanCurentLexeme(currentLine, index);
      continue;
    }

    //if we have a new lexeme, validate the type of character and set the state
    if (currentLexeme.state === LEXER_STATES.START) {
      if (REGEX.NUMBERIC.test(character)) {
        currentLexeme = new Lexeme(character, LEXER_STATES.NUMBER_ITERATION, currentLine, index);
      }
      //check if valid character
      currentLexeme = new Lexeme(character, LEXER_STATES.WORD_ITERATION, currentLine, index);
      continue;
    }

    //if we have an identifier, just keep adding to it
    if (currentLexeme.state === LEXER_STATES.WORD_ITERATION) {
      currentLexeme.text += character;
      continue;
    }

    if (currentLexeme.state === LEXER_STATES.NUMBER_ITERATION) {
      if (REGEX.NUMBERIC.test(character)) {
        currentLexeme.text += character;
        continue;
      }
      //if we have a string while on the number state, analyse the previous number and start a new state
      tokens.push(analyzeToken(currentLexeme, index - 1));

      currentLexeme = new Lexeme(character, LEXER_STATES.WORD_ITERATION, currentLine, index);
      continue;
    }
  }

  //javascript fix for no end of line character
  tokens.push(analyzeToken(currentLexeme, maxInputLenght - 1));

  return tokens.filter(Boolean);
}
```

### O analisador de cada token

```js
function analyzeToken(currentLexer, endColumn) {
  const tokenBuilder = (type) => ({
    text: currentLexer.text,
    type,
    loc: {
      line: currentLexer.startLine,
      startColumn: currentLexer.startCol,
      endColumn: endColumn + 1,
    },
  });

  if (
    currentLexer.text == undefined ||
    currentLexer.text === '' ||
    END_OF_WORD.includes(currentLexer.text) ||
    END_OF_LINE.includes(currentLexer.text)
  ) {
    return null;
  }

  if (KEYWORDS.includes(currentLexer.text)) {
    return tokenBuilder(TOKEN_TYPES.RESERVED);
  }

  if (PUNCTUATIONS.includes(currentLexer.text)) {
    return tokenBuilder(TOKEN_TYPES.PUNCTUATION);
  }

  if (REGEX.NUMBERIC.test(currentLexer.text)) {
    return tokenBuilder(TOKEN_TYPES.NUMBER);
  }

  if (REGEX.ALPHABETIC.test(currentLexer.text)) {
    return tokenBuilder(TOKEN_TYPES.IDENTIFIER);
  }

  return tokenBuilder(TOKEN_TYPES.INVALID_TOKEN);
}
```
