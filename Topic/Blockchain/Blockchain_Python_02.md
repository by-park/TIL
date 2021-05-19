# Blockchain

- 사용한 것들: python, flask, visual studio code
- 유튜브 강의 (coinsight): https://www.youtube.com/watch?v=Gno15LgVbcc
- 유튜브 강의 소스 코드: https://github.com/tr0y-kim/ez_blockchain
- 원본 강의 (hackernoon): https://hackernoon.com/learn-blockchains-by-building-one-117428612f46
- 작성한 내 결과물: https://github.com/BY1994/Blockchain



## Blockchain.py 추가될 내용

1. Register: 새로운 노드 가입
2. Consensus: 어떤 체인이 맞는 것인지에 대해 합의 (길이가 긴 체인이 맞는 체인)



새로운 노드를 만들 때는 다른 포트를 사용한다. (5001번, 5002번 등등...) 실행시켜서 각 노드들 register 해서 체인을 비교시킬 것이다.



### \__init__ 생성자내부 수정

```python
    def __init__(self):
        self.chain = [] # chain에 여러 block들 들어옴
        self.current_transaction = [] # 임시 transaction 넣어줌
        self.nodes = set() # Node 목록을 보관

        # genesis block 생성
        self.new_block(previous_hash=1, proof=100)
```

같은 노드일 경우 한 번만 저장하게 만드는 코드. 이걸 위해 set 라는 자료형태를 사용함



### register_node 추가

```python
    def register_node(self, address): # url 주소를 넣게 됨
        parsed_url = urlparse(address)
        self.nodes.add(parsed_url.netloc) # set 자료형태 안에 목록을 저장
```



### 체인 비교

체인을 비교하는 과정은 다음 2개의 함수로 구현

- valid_chain
- resolve_conflicts

한 노드들이 다른 노드와 다른 체인을 가지고 있을 때 "가장 긴 체인이 유효하다"



### valid_chain 함수 구현

```python
    def valid_chain(self, chain):
        last_block = chain[0] # 첫번째 제네시스 블록이 저장됨
        current_index = 1

        while current_index < len(chain): # 전체 체인 길이만큼 반복해 비교
            block = chain[current_index]
            print('%s', % last_block)
            print('%s', % block)
            print("\n--------\n")
            # check that the hash of the block is correct
            # hash 값을 비교해서 같지 않으면 거짓을 반환
            if block['previous_hash'] != self.hash(last_block):
                return False
            last_block = block
            current_index += 1
        return True
```



### resolve_conflicts 함수 구현

```python
    def resolve_conflicts(self):
        neighbours = self.nodes # 구동되는 노드들을 저장
        new_chain = None

        max_length = len(self.chain) # Our chain length
        for node in neighbours:
            tmp_url = "http://" + str(node) + '/chain' # url을 받아서 request 통해 체인 정보 저장
            response = requests.get(tmp_url)
            if response.status_code == 200: # 정상적으로 웹페이지와 교류가 되면 그 정보 저장
                length = response.json()['length']
                chain = response.json()['chain']

                if length > max_length and self.valid_chain(chain): # 긴 체인을 비교
                    max_length = length

            if new_chain:
                self.chain = new_chain
                return True

            return False
```



## Sever.py (웹서버) 추가될 내용

### /nodes/register

```python
@app.route('nodes/register', methods=['POST'])
def register_nodes():
    values = request.get_json() # json 형태로 보내면 노드가 저장이 됨

    nodes = values.get('nodes')
    if nodes is None: # Bad Request 400
        return "Error: Please supply a valid list of nodes", 400

    for node in nodes:
        blockchain.register_node(node) # 아까 작성한 register node 함수를 사용할 것. 노드 등록
    
    response = {
        'message' : 'New nodes have been added',
        'total_nodes' : list(blockchain.nodes),
    }
    return jsonify(response), 201
```



### /nodes/resolve

```python
@app.route('/nodes/resolve', methods='GET')
def consensus():
    replaced = blockchain.resolve_conflicts() # True Flase return

    # 체인 변경 알림 메시지
    if replaced:
        response = {
            'message' : 'Our chain was replaced',
            'new_chain' : blockchain.chain
        }
    else:
        response = {
            'message' : 'Our chain is authoritative',
            'chain' : blockchain.chain
        }
    return jsonify(response), 200
```



## 실행

5000 번 포트와 5001번 포트를 동시 실행

5000번 포트에 다음과 같이 정보 전달

```shell
$ curl -X POST -H "Content-Type: application/json" -d '{
	"nodes" : "192.168.6.128:5001"}' "http://localhost:5000/nodes/register" {"message" :"New nodes have been added", "total_nodes":[""]}
```

