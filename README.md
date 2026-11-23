import math


def ler_inteiro(mensagem, minimo=1):
    while True:
        try:
            valor = int(input(mensagem))
            if valor < minimo:
                print(f"Erro: informe um valor maior ou igual a {minimo}.")
                continue
            return valor
        except ValueError:
            print("Erro: informe um número inteiro válido.")


def ler_numero(mensagem):
    while True:
        entrada = input(mensagem).strip().replace(",", ".")
        try:
            valor = float(entrada)
            if not math.isfinite(valor):
                print("Erro: informe um número finito válido.")
                continue
            return valor
        except ValueError:
            print("Erro: informe um número válido.")


def cadastrar_matriz(numero):
    linhas = ler_inteiro(f"Quantidade de linhas da matriz {numero}: ")
    colunas = ler_inteiro(f"Quantidade de colunas da matriz {numero}: ")

    matriz = []

    print(f"\nPreenchimento da matriz {numero}:")
    for i in range(linhas):
        linha = []
        for j in range(colunas):
            valor = ler_numero(f"Elemento [{i + 1},{j + 1}]: ")
            linha.append(valor)
        matriz.append(linha)

    print()
    return matriz


def formatar_numero(valor):
    if abs(valor) < 1e-12:
        valor = 0.0

    if valor.is_integer():
        return str(int(valor))

    return f"{valor:.6f}".rstrip("0").rstrip(".")


def exibir_matriz(matriz):
    if not matriz:
        print("[]")
        return

    formatada = [
        [formatar_numero(valor) for valor in linha]
        for linha in matriz
    ]
    largura = max(len(item) for linha in formatada for item in linha)

    for linha in formatada:
        print("[", "  ".join(item.rjust(largura) for item in linha), "]")


def linhas_e_colunas(matriz):
    return len(matriz), len(matriz[0])


def adicionar(a, b):
    linhas_a, colunas_a = linhas_e_colunas(a)
    linhas_b, colunas_b = linhas_e_colunas(b)

    if (linhas_a, colunas_a) != (linhas_b, colunas_b):
        print("Erro: adição impossível. As matrizes precisam ter as mesmas dimensões.")
        return None

    return [
        [a[i][j] + b[i][j] for j in range(colunas_a)]
        for i in range(linhas_a)
    ]


def subtrair(a, b):
    linhas_a, colunas_a = linhas_e_colunas(a)
    linhas_b, colunas_b = linhas_e_colunas(b)

    if (linhas_a, colunas_a) != (linhas_b, colunas_b):
        print("Erro: subtração impossível. As matrizes precisam ter as mesmas dimensões.")
        return None

    return [
        [a[i][j] - b[i][j] for j in range(colunas_a)]
        for i in range(linhas_a)
    ]


def multiplicar(a, b):
    linhas_a, colunas_a = linhas_e_colunas(a)
    linhas_b, colunas_b = linhas_e_colunas(b)

    if colunas_a != linhas_b:
        print(
            "Erro: multiplicação impossível. "
            "O número de colunas da primeira matriz deve ser igual "
            "ao número de linhas da segunda matriz."
        )
        return None

    return [
        [
            sum(a[i][k] * b[k][j] for k in range(colunas_a))
            for j in range(colunas_b)
        ]
        for i in range(linhas_a)
    ]


def inversa(matriz):
    linhas, colunas = linhas_e_colunas(matriz)

    if linhas != colunas:
        print("Erro: divisão matricial impossível. A segunda matriz precisa ser quadrada.")
        return None

    n = linhas
    aumentada = [
        matriz[i][:] + [1.0 if i == j else 0.0 for j in range(n)]
        for i in range(n)
    ]

    for coluna in range(n):
        linha_pivo = max(
            range(coluna, n),
            key=lambda i: abs(aumentada[i][coluna])
        )

        if abs(aumentada[linha_pivo][coluna]) < 1e-12:
            print("Erro: divisão matricial impossível. A matriz não é invertível.")
            return None

        if linha_pivo != coluna:
            aumentada[coluna], aumentada[linha_pivo] = (
                aumentada[linha_pivo],
                aumentada[coluna]
            )

        pivo = aumentada[coluna][coluna]
        aumentada[coluna] = [valor / pivo for valor in aumentada[coluna]]

        for i in range(n):
            if i == coluna:
                continue

            fator = aumentada[i][coluna]
            aumentada[i] = [
                aumentada[i][j] - fator * aumentada[coluna][j]
                for j in range(2 * n)
            ]

    return [linha[n:] for linha in aumentada]


def dividir(a, b):
    inversa_b = inversa(b)
    if inversa_b is None:
        return None
    return multiplicar(a, inversa_b)


def selecionar_matrizes(matrizes):
    print("\nMatrizes cadastradas:")
    for i, matriz in enumerate(matrizes, start=1):
        linhas, colunas = linhas_e_colunas(matriz)
        print(f"{i} - Matriz {i} ({linhas}x{colunas})")

    while True:
        indice_a = ler_inteiro("Informe o número da primeira matriz: ")
        indice_b = ler_inteiro("Informe o número da segunda matriz: ")

        if 1 <= indice_a <= len(matrizes) and 1 <= indice_b <= len(matrizes):
            return matrizes[indice_a - 1], matrizes[indice_b - 1]

        print("Erro: escolha números correspondentes às matrizes cadastradas.")


def mostrar_menu():
    print("\n===== OPERAÇÕES =====")
    print("1 - Adição")
    print("2 - Subtração")
    print("3 - Multiplicação")
    print("4 - Divisão matricial (A × B⁻¹)")
    print("5 - Encerrar")


def main():
    print("===== CADASTRO DE MATRIZES =====")

    quantidade = ler_inteiro(
        "Quantidade de matrizes (mínimo 2): ",
        minimo=2
    )

    matrizes = []

    for i in range(1, quantidade + 1):
        matriz = cadastrar_matriz(i)
        matrizes.append(matriz)
        print(f"Matriz {i} cadastrada:")
        exibir_matriz(matriz)
        print()

    while True:
        mostrar_menu()

        try:
            opcao = int(input("Selecione uma operação: "))
        except ValueError:
            print("Erro: informe um número válido.")
            continue

        if opcao == 5:
            print("Programa encerrado.")
            break

        if opcao not in (1, 2, 3, 4):
            print("Erro: opção inválida.")
            continue

        a, b = selecionar_matrizes(matrizes)

        if opcao == 1:
            resultado = adicionar(a, b)
        elif opcao == 2:
            resultado = subtrair(a, b)
        elif opcao == 3:
            resultado = multiplicar(a, b)
        else:
            resultado = dividir(a, b)

        if resultado is not None:
            print("\nResultado:")
            exibir_matriz(resultado)


if __name__ == "__main__":
    main()
