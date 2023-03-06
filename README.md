# AI-generator_solidity-security-fixer

import subprocess
import json

def compile_contract(contract_path):
    command = ['solc', '--ast-compact-json', contract_path]
    result = subprocess.run(command, capture_output=True, text=True)
    if result.returncode == 0:
        ast = json.loads(result.stdout)['children']
        return ast
    else:
        print('Error compiling contract')
        return None

def lint_contract(ast):
    command = ['solhint', '--max-warnings', '0', '-f', 'json']
    for child in ast:
        if child['name'] == 'ContractDefinition':
            contract_name = child['attributes']['name']
            contract_source_code = child['attributes']['source']
            break

    with open(f'{contract_name}.sol', 'w') as f:
        f.write(contract_source_code)

    command.append(f'{contract_name}.sol')

    result = subprocess.run(command, capture_output=True, text=True)
    if result.returncode == 0:
        issues = json.loads(result.stdout)
        if len(issues) > 0:
            print(f'{len(issues)} issues found:')
            for issue in issues:
                print(f'{issue["ruleId"]}: {issue["message"]}')
        else:
            print('No issues found')
    else:
        print('Error running linter')

contract_path = 'PATH_TO_CONTRACT'
ast = compile_contract(contract_path)
if ast:
    lint_contract(ast)
