
# Guia Rápido de Tags de Binding no Go com Gin e Validator.v10

Este guia fornece uma visão geral das tags de binding que podem ser usadas na construção de DTOs em Go, especialmente ao utilizar o framework **Gin** e a biblioteca de validação **go-playground/validator.v10**.

---

## 1. Sintaxe Básica das Tags de Binding

As tags de binding são definidas nas structs dos seus DTOs e seguem o formato:

```go
type DTO struct {
    Campo Tipo `binding:"regras"`
}
```

Onde **regras** é uma lista de validadores separados por vírgulas.

---

## 2. Validadores Comuns

### Obrigatoriedade

- **`required`**: O campo é obrigatório e não pode ser vazio.
  ```go
  Nome string `binding:"required"`
  ```

### Strings

- **`min`**: Comprimento mínimo da string.
  ```go
  Senha string `binding:"required,min=8"`
  ```

- **`max`**: Comprimento máximo da string.
  ```go
  Username string `binding:"required,max=15"`
  ```

- **`len`**: Comprimento exato da string.
  ```go
  Código string `binding:"len=6"`
  ```

- **`email`**: Verifica se é um email válido.
  ```go
  Email string `binding:"required,email"`
  ```

- **`url`**: Verifica se é uma URL válida.
  ```go
  Site string `binding:"required,url"`
  ```

- **`uuid`**: Verifica se é um UUID válido.
  ```go
  ID string `binding:"required,uuid"`
  ```

### Números

- **`gt`**: Maior que um valor específico.
  ```go
  Idade int `binding:"required,gt=0"`
  ```

- **`gte`**: Maior ou igual a um valor específico.
  ```go
  Saldo float64 `binding:"required,gte=0"`
  ```

- **`lt`**: Menor que um valor específico.
  ```go
  Desconto int `binding:"lt=100"`
  ```

- **`lte`**: Menor ou igual a um valor específico.
  ```go
  Nota int `binding:"lte=10"`
  ```

- **`eq`**: Igual a um valor específico.
  ```go
  Tipo int `binding:"eq=1"`
  ```

### Outros Validadores

- **`oneof`**: O valor deve ser um dos valores especificados.
  ```go
  Sexo string `binding:"required,oneof=male female"`
  ```

- **`datetime`**: Verifica se a string corresponde a um formato de data/hora específico.
  ```go
  Data string `binding:"required,datetime=2006-01-02"`
  ```

- **`contains`**: Verifica se a string contém uma substring específica.
  ```go
  Comentário string `binding:"contains=golang"`
  ```

- **`excludes`**: Verifica se a string não contém uma substring específica.
  ```go
  Senha string `binding:"required,excludes= "`
  ```

- **`alphanum`**: Verifica se a string contém apenas caracteres alfanuméricos.
  ```go
  Código string `binding:"required,alphanum"`
  ```

- **`boolean`**: Verifica se o valor é um booleano válido.
  ```go
  Ativo bool `binding:"required"`
  ```

### Estruturas Aninhadas

- **`dive`**: Usado para validar elementos dentro de slices, arrays ou maps.
  ```go
  Tags []string `binding:"required,dive,required"`
  ```

Neste exemplo, cada elemento dentro de `Tags` é obrigatório.

---

## 3. Exemplo Completo

```go
type CreateUserRequest struct {
    Nome      string    `json:"nome" binding:"required,min=3,max=50"`
    Email     string    `json:"email" binding:"required,email"`
    Senha     string    `json:"senha" binding:"required,min=8"`
    Idade     int       `json:"idade" binding:"gte=0,lte=130"`
    DataNasc  string    `json:"data_nasc" binding:"required,datetime=2006-01-02"`
    Interesses []string `json:"interesses" binding:"dive,required"`
}
```

**Explicação:**
- **Nome**: Obrigatório, mínimo de 3 caracteres e máximo de 50.
- **Email**: Obrigatório e deve ser um email válido.
- **Senha**: Obrigatória, com no mínimo 8 caracteres.
- **Idade**: Deve ser entre 0 e 130.
- **DataNasc**: Obrigatória, deve seguir o formato `YYYY-MM-DD`.
- **Interesses**: Cada item do slice deve ser obrigatório.

---

## 4. Utilizando os Validadores no Handler

```go
func CreateUser(c *gin.Context) {
    var req CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        // Extrai erros detalhados
        var ve validator.ValidationErrors
        if errors.As(err, &ve) {
            out := make([]string, len(ve))
            for i, fe := range ve {
                out[i] = fmt.Sprintf("Campo %s: %s", fe.Field(), fe.Tag())
            }
            c.JSON(http.StatusBadRequest, gin.H{"errors": out})
            return
        }
        // Erro genérico
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    // Processa a requisição...
    c.JSON(http.StatusOK, gin.H{"message": "Usuário criado com sucesso!"})
}
```

---

## 5. Dicas Adicionais

- **Mensagens de Erro Personalizadas:** Use o campo `label` nas tags para adicionar mensagens personalizadas.
  ```go
  Nome string `json:"nome" binding:"required" label:"Nome completo"`
  ```

- **Validadores Personalizados:** É possível criar validadores personalizados para atender a requisitos específicos.

- **Combinação de Validadores:** Combine múltiplas regras em um campo.
  ```go
  Código string `binding:"required,len=6,numeric"`
  ```

- **Validando Slices, Arrays e Maps:** Use `dive` para iterar sobre elementos e aplicar validadores.
  ```go
  Telefones []string `binding:"required,dive,len=10,numeric"`
  ```

- **Ignorando Campos:** Use `binding:"-"` para ignorar um campo.
  ```go
  Ignorar string `json:"ignorar" binding:"-"`
  ```

---

## 6. Referências Úteis

- [Gin - Bindings](https://gin-gonic.com/docs/examples/binding-and-validation/)
- [Go Playground Validator.v10](https://github.com/go-playground/validator)

---
