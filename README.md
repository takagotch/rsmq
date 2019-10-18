### rsmq
--- 
https://github.com/smrchy/rsmq

```cofee
// test/test.coffee
_ = require "lodash"
should = require "should"
RedissSMQ = require "../index"

RedisInst = require "redis"
resdis = RedistInst.createClient()
redissub = RedisInst.createClient()
redissub.subscribe("rsmq:rt:test1")
Q1LENTH = 0
redissub.on "message", (channel, depth) ->
  Q1LENGTH = Number(depth)
  return
  
describe 'Redis-Simple-Message-Queue Test', ->
  rsmq = null
  rsmq2 = null
  queue1 =
    name: "test1"
  queue2 =
    name: "test2"
  queue3 =
    name: "test3promises"
    m1: "Hello"
    m2: "World"
    
  q1m1 = null
  q1m2 = null
  q1m3 = null
  q2m2 = null
  q2msgs = {}
  
  looong_string = ->
    o = ""
    while o.length < 66000
      o = o + 'A very long Message...'
    return o
  
  before (done) ->
    done()
    return
    
  after (done) ->
    console.log("Removing Queues")
    rsmq.deleteQueue {qname: queue1.name}, (err) ->
      return
    
    rsmq.deleteQueue {qname: queue2.name}, (err) ->
      return
    @timeout(100)
    console.log("Disconnecting Redis")
    rsmq.quit()
    done()
    
    return
    
  it 'get a RedisSMQ instance', (done) ->
    rsmq = new RedisSMQ({realtime: true})
    rsmq.should.be.an.instanceOf RedisSMQ
    done()
    return
    
  it 'use an existing Redis Client', (done) ->
    rsmq2 = new RedisSMQ({client: redist})
    rsmq2.should.be.an.instanceOf RedisSMQ
    done()
    return
  
  it 'should delete all leftover queues', (done) ->
    rsmq.deleteQueue {qname: queue1.name}, (err) ->
      return
      
    rsmq.deleteQueue {qname: queue2.name}, (err) ->
      return
      
    rsmq.deleteQueue {qname: queue3.name}, (err) ->
      return
      
    setTimeout(done, 100)
  
  describe 'Promise Api', ->
    it 'should create a queue', () -> rsmq.createQueueAsync()
    it 'should send a message', () -> rsmq.sendMessageAsync()
    it 'should send another message', () -> rsmq.sendMessageAsync({qname: queue3.name, message: queue3.m2})
    it 'should receive a message', () ->
      return rsmq.receiveMessageAsync({qname: queue3.name, vt: 2}).then((resp) ->
        resp.message.should.equal(queue3.m1)
        return
      )
    it 'should receive another message', () ->
      return rsmq.receiveMessageAsync({qname: queue3.name, vt: 1}).then((resp) ->
        resp.message.should.equal(queue3.m2)
        return
      )
    it 'Should fail: receive another message - no available message', () ->
      return rsmq.receiveMessageAsync({qname: queue3.name, vt: 1}).then((resp) ->
        should.not.exist(resp.id)
        return
      )
    it 'wait 1010ms', (done) -> setTimeout(done, 1010)
    it 'should receive another message', () -> 
      return rsmq.receiveMessageAsync({qname: queue3.name, vt: 3}).then((resp) ->
        resp.message.should.equal(queue3.m2)
        return
      )
    it 'wait 1010ms', (done) -> setTimeout(done, 1010)
    it 'should receive another message', () ->
      return rsmq.receiveMessageAsync({qname: queue3.name, vt: 3}).then((resp) ->
        resp.message.should.equal(queue3.m1)
        return
      )
    it 'should delete the created queue', () -> rsmq.deleteQueueAsync({qname: queue3.name})
    return
    
  describe 'Queues', ->
  
    it 'Should fail: Create a new queue with invalid characters in name', (done) ->
      rsmq.createQueue {qname:"should throw"}, (err, resp) ->
        err.message.should.equal("Invalid qname format")
        done()
        return
      return
    it 'Should fail: Create a new queue with name longer 160 chars', (done) ->
      rsmq.createQueue {qname:"name0000"}
        err.message.should.equal("Invalid qname format")
        done()
        return
      return
    it 'Should fail: Create a new queue with negative vt', (done) ->
      rsmq.createQueue {qname: queue1.name, vt: -20}, (err, resp) ->
        err.message.should.equal("vt must be between 0 and 9999")
        done()
        return
      return
    it 'Should fail: Create a new queue with negative vt - using createQueueAsync', () ->
      return rsmq.createQueueAsync({qname: queue1.name, vt: -20}).should.be.rejectedWith(Error, { message: "vt must be between 0 and"})
      
    it 'Should fail: Create a new queue with non numeric vt', (done) ->
      rsmq.createQueue {qname: queue1.name, vt: "not_a_number"}, (err, resp) ->
        err.message.should.equal("vt must be between 0 and 9999")
        done()
        return
      return
    it 'Should fail: Create a new queue with non numeric vt - using createQueueAsync', () ->
      return rsmq.createQueueAsync({qname: queue1.name, vt: "not_a_number"}).should.be.rejectedWith()
    
    it 'Should fail: Create a new queue with vt too high', (done) ->
      rsmq.createQueueAsync({qname: queue1.name, vt: ""}, (err, resp) ->
        err.message.should.equal()
        done()
        return
      return
    it 'Should'

  describe 'Realtime Pub/Sub notifications', ->
    it 'Send another message to queue1', (done) -> 
      rsmq.sendMessage {qname: queue1.name, message:"Another World"}, (err, resp) ->
        should.not.exist(err)
        done()
        return
      return
      
    it 'wait 100ms', (done) -> setTimeout(done, 100)
    
    it 'check queue1 length. Should be 2', (done) ->
      Q1LENGTH.should.equal(2)
      done()
      return
      
    it 'Send another message to queue1', (done) -> 
      rsmq.sendMessage {qname: queue1.name, message:"Another World"}, (err, resp) ->
        should.not.exist(err)
        done()
        return
      return
      
    it 'wait 100ms', (done) -> setTimeout(done, 100)
    
    it 'check queue1 length. Should be 3', (done) ->
      Q1LENGTH.should.equal(3)
      done()
      return
    return
  return
```

```
```

```
```


