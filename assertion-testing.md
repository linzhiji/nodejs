# 断言测试Assert

    稳定性: 5 - 锁定

断言（Assert）模块用于为应用编写单元测试，可以通过 `require('assert')` 对该模块进行调用。


## assert.fail(actual, expected, message, operator)

使用指定操作符测试 `actual` (真实值) 和 `expected` （期望值）一致。

## assert(value[, message]), assert.ok(value[, message])

测试实际值是否为`true`,和`assert.equal(true, !!value, message);`作用一致。

## assert.equal(actual, expected[, message])

判断实际值`actual`和期望值`expected`是否浅层`shallow`一致。

## assert.notEqual(actual, expected[, message])

判断实际值`actual`和期望值`expected`是否浅层`shallow`不等。

## assert.deepEqual(actual, expected[, message])

判断实际值`actual`和期望值`expected`是否深层`deep`一致。

## assert.notDeepEqual(actual, expected[, message])

判断实际值`actual`和期望值`expected`是否深层`deep`不等。

## assert.strictEqual(actual, expected[, message])

使用严格相等操作符 ( `===` )测试真实值是否严格地（strict）和预期值相等。
## assert.notStrictEqual(actual, expected[, message])

使用严格不相等操作符 ( `!==` )测试真实值是否严格地（strict）和预期值不相等。


## assert.throws(block[, error][, message])
预期`block`时抛出一个错误（`error`）， `error`可以为构造函数，正则表达式或者其他验证器。

使用构造函数验证实例：

    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      Error
    );

使用正则表达式验证错误信息：

    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      /value/
    );

用户自定义的错误验证器：

    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      function(err) {
        if ( (err instanceof Error) && /value/.test(err) ) {
          return true;
        }
      },
      "unexpected error"
    );

## assert.doesNotThrow(block[, message])

预期`block时`不抛出错误，详细信息请见 `assert.throws`。

## assert.ifError(value)

测试值是否不为false，当为true时抛出。常用于回调中第一个参数error的测试。
