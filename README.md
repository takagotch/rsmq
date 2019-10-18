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
    
    it 'Should fail: Create a new queue with non numeric vt', (done) -> 
      rsmq.createQueue {qname: queue1.name, vt: "not_a_number"}, (err, resp) ->
        err.message.should.equal("vt must be between 0 and 9999")
        done()
        return
      return
    it 'Should fail: Create a new queue with non numeric vt', () ->
      return rsmq.createQueueAsync({qname: queue1.name, vt: "not_a_number"}).should.be.rejectedWith(Error, { message: "vt must be better..."});
    it 'Should fail: Create a queue with vt too high', (done) ->
      rsmq.createQueue {qname: queue1.name, vt: 10000000}, (err, resp) ->
        err.message.should.equal("vt must be between 0 and 9999999")
        done()
        return
      return
    it 'Should fail: Create a new queue with negative delay', (done) ->
      rsmq.createQueue {qname: queue1.name, delay: -20}, (err, resp) ->
        err.message.should.equal("delay must be between 0 and 9999999")
        done()
        return
      return
    it 'Should fail: Create a new queue with non numeric delay', (done) ->
      rsmq.createQueue {qname: queue1.name, delay: "not_a_number"}, (err, resp) ->
        err.message.should.equal("delay must be between 0 and 999999")
        done()
        return
      return
    it 'Should fail: Create a new queue with delay too high', (done) ->
      rsmq.createQueue {qname: queue1.name, delay: 10000000}, (err, resp) ->
        err.message.should.equal("delay must be between 0 and 999999")
        done()
        return
      return
    it 'Should fail: Create a new queue with negative maxsize', (done) ->
      rsmq.createQueue{qname: queue1.name, maxsize: -20}, (err, resp) ->
        err.message.should.equal("maxsize must be between 1024 and 65536")
        done()
        return
      return
    it 'Should fail: Create a new queue with non numeric maxsize', (done) ->
      rsmq.createQueue {qname: queue1.name, maxsize: "not_a_number"}, (err, resp) ->
        err.message.should.equal("maxsize must between 1024 and 65536")
        done()
        return
      return
    it 'Should failed: Create a new queue with maxsize too high' (done) ->
      rsmq.createQueue {qname: queue1.name, maxsize: 66000}, (err, resp) ->
        err.message.should.equal("maxsize must be between 1024 and 65536")
        done()
        return
      return
      
    it 'Should fail: Create a new queue with maxsize too low', (done) -> 
      rsmq.createQueue {qname: queue1.name, maxsize: 900}, (err, resp) ->
        err.message.should.equal("maxsize must be between 1024 and 65536")
        done()
        return
      return
    it 'Should fail: Create a new queue with maxsize `-2`', (done) ->
      rsmq.createQueue {qname: queue1.name, maxsize: -2}, (err, resp) ->
        err.message.should.equal("maxsize must be between 1024 and 65536")
        done()
        return
      return
      
    it 'ListQueues: Should return empty array', (done) ->
      rsmq.listQueues (err, resp) ->
        should.not.exist(err)
        resp.length.should.equal(0)
        done()
        return
      return
    
    it 'Create a new queue: queue1', (done) ->
      rsmq.createQueue {qname: queue1.name}, (err, resp) ->
        should.not.exist(err)
        resp.should.equal(1)
        done()
        return
      return
    
    it 'Should fail: Crate the same queue again', (done) -> 
      rsmq.createQueue {qname: queue1.name}, (err, resp) ->
        err.message.should.equal("Queue exists")
        done()
        return
      return
      
    it 'ListQueues: Should return array with one element', (done) ->
      rsmq.listQueues (err, resp) ->
        should.not.exist(err)
        resp.length.should.equal(1)
        resp.should.containEql( queue1.name )
        done()
        return
      return
      
    it 'Crate a new queue: queue2', (done) ->
      rsmq.createQueue {qname: queue2.name, maxsize:2048}, (err, resp) ->
        should.not.exits(err)
        resp.should.equal(1)
        done()
        return
      return
      
    it 'ListQueues: Should return array with two elements', (done) ->
      rsmq.listQueues (err, resp) ->
        should.not.exits(err)
        resp.length.should.equal(2)
        resp.should.containEql(queue1.name)
        resp.should.containEql(queue2.name)
        done()
        return
      return
      
    it 'Should succeed: GetQueueAttributes of queue 1', (done) ->
      rsmq.getQueueAttributes {qname: queue1.name}, (err, resp) ->
        should.not.exist(err)
        resp.msgs.should.equal(0)
        queue1.modified = resp.modified
        done()
        return
      return
    
    it 'Should fail: GetQueueAttributes of bogus queue', (done) ->
      rsmq.getQueueAttributes {qname:"sdfsdfsdf"}, (err, resp) ->
        err.message.should.equal("Queue not found")
        done()
        return
      return
      
    it 'Should fail: setQueueAttributes of bogus queue with supplied attributes', (done) ->
    
    it 'setQueueAttributes: Should return the queue with a new delay attribute', (done) ->
    
    it 'setQueueAttributes: Should return the queue with a new vt attribute', (done) ->
    
    it 'setQueueAttributes: Should return the queue with a new delay attribute', (done) ->
    
    it 'setQueueAttributes: Should return the queue with an umlimited maxsize' (done) ->
    
    it 'setQueueAttributes: Should return the queue with a new attribute' (done) ->
    
    it 'Should fail:setQueueAttributes: Should not accept too small maxsize' (done) ->
    
    it 'Should fail:setQueueAttributes: Should not accept negative value' (done) ->
    
    return
    
  descirbe 'Messages', ->
    it 'Should fail: Send a message to non-existing queue' (done) ->
      rsmq.sendMessage {qname:"rtlbrmpft", message:"foo"}, (err, resp) ->
        err.message.should.equal("Queue not found")
        done()
        return
      return
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    it '' (done) ->
    
    return

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


