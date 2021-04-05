---
title: FreeNOS学习笔记(4)-C++工厂模式
date: 2021-04-05 14:36:04
tags: FreeNOS笔记
categories: FreeNOS
---
一、什么是工厂模式
=======
&emsp;&emsp;这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。<!-- more --> 简单来说，工厂模式使用了C++多态的特性，将存在继承关系的类，通过一个工厂类创建对应的子类（派生类）对象。在项目复杂的情况下，可以便于子类对象的创建。

&emsp;&emsp;本文介绍FreeNOS中一种工厂模式的设计方法，并通过实例来学习工厂模式。

二、工厂模式模板类实现
=======
&emsp;&emsp;工厂模式模板类设计为一个头文件，具体内容**Factory.h**如下： 

    /**
     * 工厂设计模式
     */
    #ifndef _FACTORY_H
    #define _FACTORY_H

    /**
     * 工厂设计模式：提供一个标准的创建函数
     */
    template <class T> class Factory
    {
    public:
        static T* create()
        {
            return new T();
        }
    };

    /**
     * 抽象工厂设计模式：提供一个创建函数定义
     */
    template <class T> class AbstractFactory
    {
    public:
        static T* create();
    };

    #endif

三、测试代码
=======
    #include <iostream>
    #include "Factory.h" 

    using namespace std;

    /**
     * 抽象产品角色 
     */
    class MobilePhone
    {
    public:
	    virtual ~MobilePhone()
	    {
		
	    };
	    virtual void show() = 0;
    };

    /**
     * 具体产品角色1 
     */
    class HuaweiMobilePhone : public MobilePhone
    {
    public:
	    HuaweiMobilePhone()
	    {
		    cout << "华为手机已创建" << endl;
	    }
		~HuaweiMobilePhone()
		{
			cout << "华为手机已报废" << endl;
		}	
		void show()
		{
			cout << "我是华为手机" << endl;
		}
	};

	/**
	 * 具体产品角色2 
	 */
	class XiaomiMobilePhone : public MobilePhone
	{
	public:
		XiaomiMobilePhone()
		{
			cout << "小米手机已创建" << endl;
		}
		~XiaomiMobilePhone()
		{
			cout << "小米手机已报废" << endl;
		}	
		void show()
		{
			cout << "我是小米手机" << endl;
		}
	};

	/**
	 * 具体手机工厂1
	 * 缺点：增加产品时，这里还要增加新的创建方法
	 */ 
	class MobilePhoneFactory : public Factory<MobilePhoneFactory>
	{
	public:
		MobilePhoneFactory()
		{
			cout << "手机工厂已创建" << endl;	
		}
		~MobilePhoneFactory()
		{
			cout << "手机工厂已破产" << endl;
		}
		
		// 提供创建产品方法 
		MobilePhone* createHuaweiMobilePhone()
		{
			return new HuaweiMobilePhone();
		}
		MobilePhone* createXiaomiMobilePhone()
		{
			return new XiaomiMobilePhone();
		}
	};
	
	/**
	 * 具体高级工厂2
	 * 特点：增加产品时，无需修改这里
	 * @param T1 产品抽象类
	 * @param T2 产品具体类 
	 */ 
	template <class T1, class T2> class AdvanceFactory : public Factory<AdvanceFactory<T1, T2>>
	{
	public:
		AdvanceFactory()
		{
			cout << "高级工厂已创建" << endl;
		}
		~AdvanceFactory()
		{
			cout << "高级工厂已破产" << endl;
		}
		T1* createProduct()
		{
			return new T2;
		} 
	}; 
	typedef AdvanceFactory<MobilePhone, HuaweiMobilePhone> HuaweiFactory_t;
	typedef AdvanceFactory<MobilePhone, XiaomiMobilePhone> XiaomiFactory_t;	 
	
	int main(int argc, char** argv) 
	{
		// 创建工厂 
		MobilePhoneFactory *pMobilePhoneFactory = MobilePhoneFactory::create();
		
		// 创建和报废产品1 
		MobilePhone *pMobilePhone1 = pMobilePhoneFactory->createHuaweiMobilePhone();
		pMobilePhone1->show();
		delete pMobilePhone1;
		
		// 创建和报废产品2 
		MobilePhone *pMobilePhone2 = pMobilePhoneFactory->createXiaomiMobilePhone();
		pMobilePhone2->show();
		delete pMobilePhone2;
		
		// 工厂破产 
		delete pMobilePhoneFactory;
		
		/********************************分割线**********************************/
		
		HuaweiFactory_t *pHuaweiFactory = HuaweiFactory_t::create();
		MobilePhone *pMobilePhone3 = pHuaweiFactory->createProduct();
		pMobilePhone3->show();
		delete pMobilePhone3;
		delete pHuaweiFactory;
		
		XiaomiFactory_t *pXiaomiFactory = XiaomiFactory_t::create();
		MobilePhone *pMobilePhone4 = pXiaomiFactory->createProduct();
		pMobilePhone4->show();
		delete pMobilePhone4;
		delete pHuaweiFactory;
	}