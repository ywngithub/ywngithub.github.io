---
title: FreeNOS学习笔记(3)-C++单例模式
date: 2021-04-04 21:24:09
tags: FreeNOS笔记
categories: FreeNOS
---

一、什么是单例模式
=======
&emsp;&emsp;在设计或开发中，肯定会有这么一种情况，一个类只能有一个对象被创建，如果有多个对象的话，可能会导致状态的混乱和不一致。这种情况下，单例模式是最恰当的解决办法。它有很多种实现方式，各自的特性不相同，使用的情形也不相同。<!-- more -->  
&emsp;&emsp;通过单例模式，可以做到:
>1. 确保一个类只有一个实例被建立 
>2. 提供了一个对对象的全局访问指针 
>3. 在不影响单例类的客户端的情况下允许将来有多个实例

&emsp;&emsp;本文介绍FreeNOS中一种实现单例模式的方法，用户类通过继承预先设计好的单例模板类可轻松实现类的单例，与常见的单例实现方法稍微有些不同。

二、单例模板类实现
=======
&emsp;&emsp;单例模板类设计为一个头文件，具体内容**Singleton.h**如下：  

    /**
     * 单例设计模式
     */
    
    #ifndef _SINGLETON_H
    #define _SINGLETON_H

    /**
     * 单例设计模式：只有一个实例被允许
     * - 永远只有一个实例
     * - 这个实例不能被覆盖
     * - 这个实例在首次调用时创建
     * - 用户类必须提供一个没有参数的默认构造函数
     */
    template <class T> class StrictSingleton
    {
    public:
    	static inline T *instance()
    	{
    		static T obj;
    		return &obj;
    	}
    };
    
    /**
     * 单例设计模式：只有一个实例被允许
     * - 永远只有一个实例
     * - 这个实例可以被覆盖
     * - 这个实例在任何时候都可以为0
     */
    template <class T> class WeakSingleton
    {
    public:
	    /**
	     * 构造函数
	     * 
	     * @param obj 新的实例
	     */
	    WeakSingleton<T>(T* obj)
	    {
	        m_instance = obj;
	    }

	    /**
	     * 返回实例
	     */
	    static inline T *instance()
	    {
	        return m_instance;
	    }

    private:
    	/** 只有单例 */
    	static T *m_instance;
    };
    /* 初始化静态成员变量 */
    template <class T> T* WeakSingleton<T>::m_instance = 0;
    
    #endif

三、测试代码
=======
    #include <iostream>
    #include "Singleton.h" 
    
    using namespace std;
    
    class myClassStrictSingleton : public StrictSingleton<myClassStrictSingleton>
    {
    public:
    	myClassStrictSingleton() : m_count(0)
    	{
    
    	}
    	~myClassStrictSingleton()
    	{
    
    	}
    	void myFunction()
    	{
    		m_count++;
    		cout << "m_count: " << m_count << endl;
    	}
    	
    private:
    	int m_count;
    };
     
    class myClassWeakSingleton : public WeakSingleton<myClassWeakSingleton>
    {
    public:
    	myClassWeakSingleton() : WeakSingleton<myClassWeakSingleton>(this), m_count(0)
    	{
    
    	}
    	~myClassWeakSingleton() 
    	{
    				
    	}
    	void myFunction()
    	{
    		m_count++;
    		cout << "m_count: " << m_count << endl;
    	}
    	
    private:
    	int m_count;	
    };
    
    int main(int argc, char** argv) 
    {
    	myClassStrictSingleton c1;	// 非必要 
    	myClassStrictSingleton::instance()->myFunction();	// m_count = 1 
    	myClassStrictSingleton::instance()->myFunction();	// m_count = 2 
    	
    	myClassStrictSingleton c2;	// 注意：实例不会被覆盖 
    	myClassStrictSingleton::instance()->myFunction();	// m_count = 3 
    	
    	
    	myClassWeakSingleton c3;	// 第一个创建是必要的 
    	myClassWeakSingleton::instance()->myFunction();	// m_count = 1 
    	myClassWeakSingleton::instance()->myFunction();	// m_count = 2
    	
    	myClassWeakSingleton c4;	// 注意：实例将会被覆盖 
    	myClassWeakSingleton::instance()->myFunction();	// m_count = 1
    	return 0;
    }
    
