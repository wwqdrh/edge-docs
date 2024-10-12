
> 一个服务示例demo

```yaml
db:
  base:
    driver: sqlite
    dsn: ./testdata/test.db
    temp: true
    schema:
      - |
        CREATE TABLE user (
          id INTEGER PRIMARY KEY, 
          name text
        );
        INSERT INTO user (name) VALUES ("zhangsan");
        INSERT INTO user (name) VALUES ("lisi");
    alias:
      all: |
        SELECT * from user
      addone: |
        INSERT INTO user (name) VALUES (:name)
middlewares:
  local:
    id: local
  ipLimit:
    id: ipLimit
    opts:
      block_time: 5
services:
  adduser:
    url: /api/user/add
    method: post
    middlewares:
      - local
      - ipLimit
    request:
      data:
        - name: Name
          type: string
          mode: query
    response:
      type: application/json
      mock:
        default:
          content: '{"code": 0, "msg": "ok"}'
      script: |
        resjson(200, "hello cloard");
  getuser:
    url: /api/user/get
    method: get
    middlewares:
      - local
      - ipLimit
    request:
      data:
        - name: Name
          type: string
          mode: query
    response:
      type: application/json
      script: |
        state("base", "all");
        var name = req('Name', 'string');
        resjson(200, name);
      mock:
        default:
          content: '{"code": 0, "msg": "ok", "data": [{"name": "user1"}]}'
```