# 题目 - asyncPool
请实现一个名为 asyncPool 的函数，它接收三个参数：

poolLimit (number): 并发限制数，表示最多可以同时执行的异步任务数量。

array (Array): 一个包含任务数据的数组。

iteratorFn (Function): 一个异步函数，接收 array 中的一个元素作为参数，并返回一个 Promise。

asyncPool 函数需要遍历 array 中的每一项，并使用 iteratorFn 来处理它，但要确保任何时候正在执行的异步任务（pending状态的Promise）数量不能超过 poolLimit。当所有任务都执行完毕后，asyncPool 应该返回一个 Promise，该 Promise resolve 的值是一个包含了所有任务结果的数组（结果的顺序需要和输入 array 的顺序保持一致）。

请写出这个函数的实现。


# 思路

维护一个执行中的任务池 executing (一个 Set 或数组)。

维护一个结果数组 results，用于按顺序存放结果。

从 array 中取出任务，当 executing 的数量小于 poolLimit 时，就启动一个新任务。

每当一个任务完成时，将它从 executing 中移除，并尝试启动下一个任务。

需要用 Promise.all 来等待所有任务最终完成。

# 实现
```js
async function asyncPool(poolLimit, array, iteratorFn) {
  const results =; // 存储所有任务的结果，需要保证顺序
  const executing =; // 存储正在执行的任务（Promise）
  let i = 0; // 指向下一个要执行的任务的索引

  const enqueue = () => {
    // 边界条件：所有任务都已启动
    if (i === array.length) {
      return Promise.resolve();
    }

    const item = array[i];
    const index = i;
    i++;

    // 创建一个新的异步任务
    const p = Promise.resolve().then(() => iteratorFn(item, array));
    results[index] = p; // 先占位，保证结果顺序

    // 任务完成后，从执行池中移除
    const e = p.then(() => executing.splice(executing.indexOf(e), 1));
    executing.push(e);

    // 如果执行池未满，则递归启动下一个任务
    // 否则，等待池中最先完成的任务结束后，再启动下一个
    let r = Promise.resolve();
    if (executing.length >= poolLimit) {
      r = Promise.race(executing);
    }

    return r.then(() => enqueue());
  };

  await enqueue();

  // 所有任务都已启动并记录在 results 数组中，等待它们全部完成
  return Promise.all(results);
}

// --- 测试用例 ---
const timeout = (ms) => new Promise(resolve => setTimeout(() => resolve(ms), ms));
const testFn = async () => {
  const results = await asyncPool(2, , timeout);
  console.log(results); // 期望输出 
  // 观察执行顺序：1000和5000先开始，1000结束后3000开始，3000结束后2000开始
};
testFn();
```
