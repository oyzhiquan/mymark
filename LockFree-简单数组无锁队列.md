---
title: LockFree-简单数组无锁队列
date: 2019-07-12 09:42:04
tags: C++,LockFree
---

## 简介

支持SPSC（单生产者单消费者）的无锁队列。基于数组实现，避免了多线程中的内存动态分配。

## 原理

利用C++ 11的原子量实现原子操作和CAS。

writeIndex:新元素入列时存放位置在数组中的下标。

readIndex:下一个出列元素在数组中的下标。

当readIndex小于writeIndex时，可读。

## 代码

ArrayLockFreeQueue.h

```c++
#ifndef _ARRAYLOCKFREEQUEUE_H___
#define _ARRAYLOCKFREEQUEUE_H___

#include <atomic>

using namespace std;

#define QUEUE_INT unsigned long

#define ARRAY_LOCK_FREE_Q_DEFAULT_SIZE 65535 // 2^16

template <typename ELEM_T, QUEUE_INT Q_SIZE = ARRAY_LOCK_FREE_Q_DEFAULT_SIZE>
class ArrayLockFreeQueue
{
public:

	ArrayLockFreeQueue();
	virtual ~ArrayLockFreeQueue();

	QUEUE_INT size();

	bool enqueue(const ELEM_T &a_data);

	bool dequeue(ELEM_T &a_data);

    bool try_dequeue(ELEM_T &a_data);

private:

	ELEM_T m_thequeue[Q_SIZE];

	atomic_ulong m_writeIndex;

	atomic_ulong m_readIndex;

	atomic_ulong m_maximumReadIndex;

	inline QUEUE_INT countToIndex(QUEUE_INT a_count);
};

#include "ArrayLockFreeQueueImp.h"

#endif
```

ArrayLockFreeQueueImp.h

```c++
#ifndef _ARRAYLOCKFREEQUEUEIMP_H___
#define _ARRAYLOCKFREEQUEUEIMP_H___

#include "ArrayLockFreeQueue.h"

template <typename ELEM_T, QUEUE_INT Q_SIZE>
ArrayLockFreeQueue<ELEM_T, Q_SIZE>::ArrayLockFreeQueue() :
	m_writeIndex(0),
	m_readIndex(0),
	m_maximumReadIndex(0)
{

}

template <typename ELEM_T, QUEUE_INT Q_SIZE>
ArrayLockFreeQueue<ELEM_T, Q_SIZE>::~ArrayLockFreeQueue()
{

}

template <typename ELEM_T, QUEUE_INT Q_SIZE>
inline QUEUE_INT ArrayLockFreeQueue<ELEM_T, Q_SIZE>::countToIndex(QUEUE_INT a_count)
{
	return (a_count % Q_SIZE);
}

template <typename ELEM_T, QUEUE_INT Q_SIZE>
QUEUE_INT ArrayLockFreeQueue<ELEM_T, Q_SIZE>::size()
{
	QUEUE_INT currentWriteIndex = m_writeIndex;
	QUEUE_INT currentReadIndex = m_readIndex;

	if (currentWriteIndex >= currentReadIndex)
		return currentWriteIndex - currentReadIndex;
	else
		return Q_SIZE + currentWriteIndex - currentReadIndex;

}

template <typename ELEM_T, QUEUE_INT Q_SIZE>
bool ArrayLockFreeQueue<ELEM_T, Q_SIZE>::enqueue(const ELEM_T &a_data)
{
	QUEUE_INT currentWriteIndex;
	//QUEUE_INT currentReadIndex;

	currentWriteIndex = m_writeIndex;
	//currentReadIndex = m_readIndex;

	m_thequeue[countToIndex(currentWriteIndex)] = a_data;
	m_writeIndex++;
	return true;
}

template <typename ELEM_T, QUEUE_INT Q_SIZE>
bool ArrayLockFreeQueue<ELEM_T, Q_SIZE>::try_dequeue(ELEM_T &a_data)
{
	return dequeue(a_data);
}

template <typename ELEM_T, QUEUE_INT Q_SIZE>
bool ArrayLockFreeQueue<ELEM_T, Q_SIZE>::dequeue(ELEM_T &a_data)
{
	QUEUE_INT currentMaximumReadIndex;
	QUEUE_INT currentReadIndex;
	//只支持一写一读，多读时单个线程不能读到队列的所有数
	do
	{
		currentReadIndex = m_readIndex;
		currentMaximumReadIndex = m_writeIndex;

		if (countToIndex(currentReadIndex) == countToIndex(currentMaximumReadIndex))
		{
			return false;
		}
		
		a_data = m_thequeue[countToIndex(currentReadIndex)];

		if (m_readIndex.compare_exchange_weak(currentReadIndex, (currentReadIndex + 1)))
		{
			return true;
		}
	} while (true);

	return false;
}

#endif
```

