# Case-Study-Based-Analysis-and-Design-Using-Pushdown-Automata-and-Context-Free-Grammar.

import re


# =====================================================
# 1. ARITHMETIC EXPRESSION VALIDATION USING CFG
# =====================================================

class ArithmeticValidator:

    def __init__(self, expression):
        self.expression = expression
        self.tokens = re.findall(
            r'\d+(?:\.\d+)?|[()+*/^%-]|-',
            expression
        )
        self.position = 0

    def current(self):
        if self.position < len(self.tokens):
            return self.tokens[self.position]
        return None

    def match(self, token):
        if self.current() == token:
            self.position += 1
            return True
        return False

    # Grammar:
    # E -> T ((+ | -) T)*
    # T -> P ((* | / | %) P)*
    # P -> F (^ P)?
    # F -> NUMBER | (E) | (+F) | (-F)

    def expression_rule(self):

        self.term()

        while self.current() in ("+", "-"):
            self.position += 1
            self.term()

    def term(self):

        self.power()

        while self.current() in ("*", "/", "%"):
            self.position += 1
            self.power()

    def power(self):

        self.factor()

        if self.match("^"):
            self.power()

    def factor(self):

        if self.current() in ("+", "-"):
            self.position += 1
            self.factor()

        elif self.match("("):
            self.expression_rule()

            if not self.match(")"):
                raise ValueError("Missing closing parenthesis")

        elif self.current() and re.fullmatch(
            r'\d+(?:\.\d+)?', self.current()
        ):
            self.position += 1

        else:
            raise ValueError("Invalid expression")

    def validate(self):

        if not self.tokens:
            return False

        try:
            # Ensure every input character was tokenized
            if re.sub(r'\s+', '', self.expression) != ''.join(self.tokens):
                return False

            self.expression_rule()

            return self.position == len(self.tokens)

        except (ValueError, TypeError):
            return False


def arithmetic_validation():

    print("\n===== ARITHMETIC EXPRESSION VALIDATOR =====")

    expression = input("Enter Arithmetic Expression: ")

    validator = ArithmeticValidator(expression)

    if validator.validate():
        print("Accepted: Valid Arithmetic Expression!")
    else:
        print("Rejected: Invalid Arithmetic Expression!")


# =====================================================
# 2. PROGRAMMING LANGUAGE SYNTAX CHECKER USING CFG
# =====================================================

class SyntaxChecker:

    def __init__(self, code):

        self.code = code
        self.tokens = re.findall(
            r'\d+(?:\.\d+)?|[a-zA-Z_]\w*|==|!=|<=|>=|[=+\-*/%(){};<>]',
            code
        )

        self.position = 0

    def current(self):

        if self.position < len(self.tokens):
            return self.tokens[self.position]

        return None

    def match(self, token):

        if self.current() == token:
            self.position += 1
            return True

        return False

    # Grammar:
    # Program -> Statement*
    # Statement -> Assignment | Print | If | While
    # Assignment -> ID = Expression ;
    # Print -> print ( Expression ) ;
    # If -> if ( Expression ) { Statement* }
    # While -> while ( Expression ) { Statement* }

    def parse_expression(self):

        self.parse_term()

        while self.current() in ("+", "-"):
            self.position += 1
            self.parse_term()

    def parse_term(self):

        self.parse_factor()

        while self.current() in ("*", "/", "%"):
            self.position += 1
            self.parse_factor()

    def parse_factor(self):

        token = self.current()

        if token in ("+", "-"):
            self.position += 1
            self.parse_factor()

        elif token == "(":

            self.position += 1
            self.parse_expression()

            if not self.match(")"):
                raise ValueError("Missing closing parenthesis")

        elif token is not None and (
            re.fullmatch(r'\d+(?:\.\d+)?', token)
            or re.fullmatch(r'[a-zA-Z_]\w*', token)
        ):

            self.position += 1

        else:
            raise ValueError("Invalid expression")

    def parse_block(self):

        if not self.match("{"):
            raise ValueError("Missing opening brace")

        while self.current() is not None and self.current() != "}":
            self.parse_statement()

        if not self.match("}"):
            raise ValueError("Missing closing brace")

    def parse_statement(self):

        token = self.current()

        if token == "print":

            self.position += 1

            if not self.match("("):
                raise ValueError("Expected (")

            self.parse_expression()

            if not self.match(")"):
                raise ValueError("Expected )")

            if not self.match(";"):
                raise ValueError("Expected ;")

        elif token in ("if", "while"):

            self.position += 1

            if not self.match("("):
                raise ValueError("Expected (")

            self.parse_expression()

            if not self.match(")"):
                raise ValueError("Expected )")

            self.parse_block()

        elif token is not None and re.fullmatch(
            r'[a-zA-Z_]\w*', token
        ):

            self.position += 1

            if not self.match("="):
                raise ValueError("Expected =")

            self.parse_expression()

            if not self.match(";"):
                raise ValueError("Expected ;")

        else:
            raise ValueError("Invalid statement")

    def validate(self):

        try:

            # Check that no unrecognized characters were skipped
            if re.sub(r'\s+', '', self.code) != ''.join(self.tokens):
                return False

            if not self.tokens:
                return False

            while self.current() is not None:
                self.parse_statement()

            return self.position == len(self.tokens)

        except (ValueError, TypeError):
            return False


def syntax_checking():

    print("\n===== PROGRAMMING LANGUAGE SYNTAX CHECKER =====")

    print("Supported statements:")
    print("1. Assignment: x = 10;")
    print("2. Print: print(x + 5);")
    print("3. If: if (x) { print(x); }")
    print("4. While: while (x) { x = x - 1; }")

    code = input("\nEnter Code: ")

    checker = SyntaxChecker(code)

    if checker.validate():
        print("Accepted: Valid Programming Syntax!")
    else:
        print("Rejected: Invalid Programming Syntax!")


# =====================================================
# 3. BALANCED PARENTHESES USING PUSH DOWN AUTOMATA
# =====================================================

def balanced_parentheses():

    print("\n===== BALANCED PARENTHESES CHECKER =====")

    expression = input("Enter Expression: ")

    stack = []

    pairs = {
        ')': '(',
        ']': '[',
        '}': '{'
    }

    valid = True

    for char in expression:

        if char in "([{":

            # PUSH operation
            stack.append(char)

        elif char in ")]}":

            # POP operation
            if not stack or stack[-1] != pairs[char]:
                valid = False
                break

            stack.pop()

    # Accept only if stack is empty
    if valid and len(stack) == 0:
        print("Accepted: Parentheses are Balanced!")
    else:
        print("Rejected: Parentheses are Not Balanced!")


# =====================================================
# 4. PALINDROME CHECKER USING PUSH DOWN AUTOMATA
# =====================================================

def palindrome_checker():

    print("\n===== PALINDROME CHECKER =====")

    string = input("Enter a String (a and b only): ")

    if not string or any(char not in "ab" for char in string):
        print("Rejected: Enter a non-empty string containing only a and b.")
        return

    # NPDA idea:
    # Guess the midpoint, push the first half,
    # then pop and match the second half.

    accepted = False
    n = len(string)

    # Try every possible midpoint for even-length strings
    for middle in range(n + 1):

        stack = list(string[:middle])

        right = string[middle:]

        if all(
            stack.pop() == char
            for char in right
        ) and not stack:

            accepted = True
            break

    # Try every possible middle symbol for odd-length strings
    if not accepted:

        for middle in range(n):

            stack = list(string[:middle])

            right = string[middle + 1:]

            if all(
                stack.pop() == char
                for char in right
            ) and not stack:

                accepted = True
                break

    if accepted:
        print("Accepted: It is a Palindrome!")
    else:
        print("Rejected: It is Not a Palindrome!")


# =====================================================
# 5. PASSWORD PATTERN VALIDATION USING CFG
# =====================================================

def password_pattern_validation():

    print("\n===== PASSWORD PATTERN VALIDATION =====")

    password = input("Enter Password: ")

    # Example password grammar:
    # S -> U L D R
    # U -> At least one uppercase letter
    # L -> At least one lowercase letter
    # D -> At least one digit
    # R -> At least one special character
    # Minimum total length = 8

    uppercase = False
    lowercase = False
    digit = False
    special = False

    length = 0

    for char in password:

        if not char.isascii() or not char.isprintable() or char.isspace():
            print("Rejected: Invalid Character!")
            return

        length += 1

        if char.isupper():
            uppercase = True

        elif char.islower():
            lowercase = True

        elif char.isdigit():
            digit = True

        else:
            special = True

    if (length >= 8 and uppercase and lowercase
            and digit and special):

        print("Accepted: Password Pattern Matched!")

    else:

        print("Rejected: Password Pattern Not Matched!")

        print("\nPassword Requirements:")
        print("- Minimum 8 characters")
        print("- One uppercase letter")
        print("- One lowercase letter")
        print("- One digit")
        print("- One special character")


# =====================================================
# MAIN MENU
# =====================================================

def main():

    while True:

        print("\n==============================================")
        print(" PUSH DOWN AUTOMATA AND CONTEXT-FREE GRAMMAR")
        print("==============================================")

        print("1. Arithmetic Expression Validation")
        print("2. Programming Language Syntax Checker")
        print("3. Balanced Parentheses Checker")
        print("4. Palindrome Checker")
        print("5. Password Pattern Validation")
        print("6. Exit")

        choice = input("\nEnter Your Choice: ")

        if choice == "1":

            arithmetic_validation()

        elif choice == "2":

            syntax_checking()

        elif choice == "3":

            balanced_parentheses()

        elif choice == "4":

            palindrome_checker()

        elif choice == "5":

            password_pattern_validation()

        elif choice == "6":

            print("\nThank You!")
            print("Program Terminated Successfully.")
            break

        else:

            print("Invalid Choice! Please Try Again.")


# =====================================================
# PROGRAM EXECUTION
# =====================================================

if __name__ == "__main__":

    main()
