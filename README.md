# calculator
1+1
#!/usr/bin/env python3
"""
Simple safe calculator (CLI).

Supports: +, -, *, /, %, **, parentheses, unary + and -.
Usage: run `python simple_calculator.py` and type expressions like:
  1 + 2 * (3 - 4) ** 2
Type `q` or `quit` or `exit` to leave.
"""
import ast
import operator as op

# supported operators
_BIN_OPS = {
    ast.Add: op.add,
    ast.Sub: op.sub,
    ast.Mult: op.mul,
    ast.Div: op.truediv,
    ast.Mod: op.mod,
    ast.Pow: op.pow,
}
_UNARY_OPS = {
    ast.UAdd: lambda x: x,
    ast.USub: lambda x: -x,
}

def _eval(node):
    if isinstance(node, ast.Constant):  # Python 3.8+
        if isinstance(node.value, (int, float)):
            return node.value
        raise ValueError("Unsupported constant type")
    if isinstance(node, ast.Num):  # older AST node
        return node.n
    if isinstance(node, ast.BinOp):
        left = _eval(node.left)
        right = _eval(node.right)
        op_type = type(node.op)
        if op_type in _BIN_OPS:
            return _BIN_OPS[op_type](left, right)
        raise ValueError(f"Unsupported binary operator: {op_type}")
    if isinstance(node, ast.UnaryOp):
        operand = _eval(node.operand)
        op_type = type(node.op)
        if op_type in _UNARY_OPS:
            return _UNARY_OPS[op_type](operand)
        raise ValueError(f"Unsupported unary operator: {op_type}")
    if isinstance(node, ast.Expression):
        return _eval(node.body)
    raise ValueError(f"Unsupported expression: {type(node)}")

def evaluate(expr: str):
    """
    Safely evaluate a numeric expression containing only allowed nodes/operators.
    """
    parsed = ast.parse(expr, mode="eval")
    return _eval(parsed.body)

def repl():
    print("Simple Calculator (safe Python AST evaluator)")
    print("Type expressions using + - * / % ** and parentheses. Type q to quit.")
    while True:
        try:
            s = input("calc> ").strip()
        except (EOFError, KeyboardInterrupt):
            print()
            break
        if not s:
            continue
        if s.lower() in ("q", "quit", "exit"):
            break
        try:
            result = evaluate(s)
            print(result)
        except ZeroDivisionError:
            print("Error: division by zero")
        except Exception as e:
            print("Error:", e)

if __name__ == "__main__":
    repl()
