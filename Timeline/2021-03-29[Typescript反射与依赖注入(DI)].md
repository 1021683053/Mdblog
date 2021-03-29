# Typescript 反射与依赖注入(DI)

## 前言
依赖注入的优点 依赖注入降低了依赖和被依赖类型间的耦合，在修改被依赖的类型实现时，不需要修改依赖类型的实现.
传统项目中，例如 React 项目，我们的依赖结构是树状的，依赖结构本身没问题，不过依赖查找也是由文件目录结构决定的。再一部分重写模块的时候出现，所有依赖这个模块的文件的所有引入路径都需要修改，并产生连锁反应。传统在需要继承重写的项目中，我们不但需要关心类的接口实现，也需要关心类实例化需要其他类作为依赖。
前端依赖注入主要提供一下能力
1，自动实例化注入（类不参与被依赖类的实例化, 实例化交给三方控制, 解耦类与类之间的关系）
2，可被独立替换（类可以实现接口所有重写与覆盖都基于接口开发）

## Reflect Metadata
反射源数据在 Typescript 中只是一个扩展的配置存在，需要做一定配置才可以开启
* 安装依赖 `npm i reflect-metadata —save` 
* `tsconfig.json` 配置 `experimentalDecorators` 与 `emitDecoratorMetadata` 为 `true` 开启

**可以装饰的内容**
* Class 类
* Property 属性
* Method 方法
* Parameter 参数

**四种默认实现的 metadataKey**
* 类型元数据 使用元数据键`design:type`
* 参数类型元数据 使用元数据键`design:paramtypes`
* 返回类型元数据 使用元数据键`design:returntype`

**相关文档**
- [Metadata Proposal - ECMAScript](https://rbuckton.github.io/reflect-metadata/#introduction)
- [GitHub - rbuckton/reflect-metadata](https://github.com/rbuckton/reflect-metadata)


## 注解与装饰器
* 注解（Annotation）：仅提供附加元数据支持，并不能实现任何操作。需要另外的扫描器，根据元数据执行相应操作。
* 装饰器（Decorator）：仅提供定义劫持，能够对类及其方法的定义并没有提供任何附加元数据的功能。

> 从实现功能角度，两者没有本质区别，只是专注做的事情不一样。

**相关文章**
- [装饰器与注解区别](https://blog.thoughtram.io/angular/2015/05/03/the-difference-between-annotations-and-decorators.html)

## Typescript 中的 Reflect Metadata 使用
> 由于Typescript 最终还是需要转换成为JavaScript去运行，所以Typescript中的反射实际还是再运行时。Typescript 中的 design:paramtypes 中的意义就不是很大，interface 最终显示类型为一个Object，只有入参类型是需要注入类本身，才可以获取到其实际类型。  

**作用**
* 操作因访问权限限制的属性和方法； 
* 实现自定义注解；


**Reflect Metadata 使用**
1, 基础的使用 Reflect.getMetadata 来获取 metadata
```typescript
import "reflect-metadata";

@Reflect.metadata("custom:annotation:class", "TestBasicClass")
class TestBasic {
  @Reflect.metadata("custom:annotation:property", "TestBasicName")
  public name = 123;

  @Reflect.metadata("custom:annotation:method", "TestBasicFunc")
  public func() {}
}

const test = new TestBasic();

console.log(Reflect.getMetadata("custom:annotation:class", TestBasic));
console.log(Reflect.getMetadata("custom:annotation:property", test, "name"));
console.log(Reflect.getMetadata("custom:annotation:method", test, "func"));
```

2, 基础使用 Reflect.defineMetadata 自定义一个固定值注解
```typescript
// Reflect.defineMetadata 自定义注解类型
const CUSTOM_METADATA_KEY = "CUSTOM_METADATA_KEY";
const TestAnnotation = (target: any) => {
  Reflect.defineMetadata(CUSTOM_METADATA_KEY, "自定义测试", target);
};

@TestAnnotation
class TestBasic2 {}

console.log(Reflect.getMetadata("CUSTOM_METADATA_KEY", TestBasic2));
```

3, 使用 Reflect.defineMetadata 定义一个可传参的注解
```typescript
// Reflect.defineMetadata 自定义可传参注解类型
const CUSTOM_METADATA_VALUE_KEY = "CUSTOM_METADATA_VALUE_KEY";
const TestValueAnnotation =(value: any) => (target: any) => {
  Reflect.defineMetadata(CUSTOM_METADATA_VALUE_KEY, value, target);
};

@TestValueAnnotation('自定义Value')
class TestBasic3 {}

console.log(Reflect.getMetadata("CUSTOM_METADATA_VALUE_KEY", TestBasic3));
```

从以上三个例子可以看出，Reflect 可以再属性，类本身定义一些值，再次通过metadataKey这个类去获取这些定义后的值。

4，使用 `design:paramtypes` 来获取入参类型信息
```typescript
// 使用 design:paramtypes 来获取入参类型信息 （这一段再codesandbox中不生效）
const TestParamtypesAnnotation = (target: any, key: string) => {
  var types = Reflect.getMetadata("design:paramtypes", target, key);
  console.log(types);
};

class Grade {}

class TestBasic4 {
  @TestParamtypesAnnotation
  public funcPara(name: string, age: number, grade: Grade) {}
}
```

5, 使用 获取 Params 上信息
```typescript
const TestParamAnnotation = () => (target: any, key: string, index: number) => {
  console.log(target, key, index);
};

class Grade2 {}

class TestBasic5 {
  public funcPara(
    @TestParamAnnotation() name: string,
    age: number,
    grade: Grade2
  ) {}
}
```

## 使用Typescript实现简单的依赖注入
``` typescript

type Constructor<T = any> = new (…args: any[]) => T;

// 自定义 metadataKey 用来定义注入 metadata 信息的key
const INJECTABLE_PARAM_KEY_PREFIX = 'INJECTABLE_PARAM_KEY_PREFIX';

// 可以被注入，这个需要存在的原因，只有被装饰器装饰过的类才能使用反射
const Injectable = (): ClassDecorator => target => {}; // 为什么要有这个有特殊原因

// 通过构造函数的入参，根据 string 类型的 id 来注入信息
const Inject = (id) => (…args)=>{
  const [target, key, index] = args;
  const value = { key, id, index }
  Reflect.defineMetadata(INJECTABLE_PARAM_KEY_PREFIX+':'+index, value, target);
}

// Warrior Class 的接口
interface WarriorInterface {
  fight(): string;
}

// 类 Warrior 基于接口 WarriorInterface
class Warrior implements WarriorInterface{
  fight(){
    return 'fight'
  }
}

// 类 Weapon 无接口
@Injectable()
class Weapon{

  // 这里注入了另一个依赖
  constructor(@Inject('Warrior') public readonly warrior: WarriorInterface){}
  hit(){
    return 'hit'
  }
  print(){
    console.log( this.warrior.fight() )
  }
}


@Injectable()
class Service {
  // 第一个参数的类型是Object 第二个参数的类型为Weapon类 所以注入的时候要区别对待
  constructor(@Inject('Warrior') public readonly warrior: WarriorInterface, public readonly weapon: Weapon) {}
  public print (){
    console.log( this.weapon.hit() )
    this.weapon.print()
  }
}

// DI 容器，存放所有绑定的类与对应的标识
class Container {
  public targets = new Map();

  // 绑定到当前容器
  bind(id: any, target?: Constructor){
    if( !target ) return this.targets.set(id, id);
    this.targets.set(id, target);
  }

  // 获取实例
  get(id){
    const target = this.targets.get(id);
    
    // 获取所有的参数类型
    const providers = Reflect.getMetadata('design:paramtypes', target);
    
    // 判断无参数或者参数为实例化
    if( !providers || !providers.length ) return new target();

    // 实例化每个入参
    const args = providers.map((provider, index)=>{
      
      // 这里简化判断其实还有更多类型 例如 Number，String等等
      if( provider === Object ){ 

        // 获取所有 metadataKey
        const metadataKeys = Reflect.getMetadataKeys(target); 

        // 查找当前索引的metadataKey
        const metadataKey = metadataKeys.find((key)=> key === INJECTABLE_PARAM_KEY_PREFIX+':'+index );

        // 查找不到直接抛错，未找到依赖
        if( !metadataKey ) throw new Error('not fond dependency');

        // 获取入参对应metadataKey的meta信息
        const metadata = Reflect.getMetadata(metadataKey, target);

        // 根据metadata 信息获取需要注入的Class
        return this.get(metadata.id); // 这里使用递归是可能有多层依赖
      }

      // 如果类型是Class 直接获取到实例
      return this.get(provider); // 这里使用递归是可能有多层依赖
    });

    // 创建实例，并传入构造完成的入参
    return new target(…args);
  }
}

// 初始化容器
const container = new Container();

// 绑定 Class 基于 id 为 string
container.bind('Warrior', Warrior);

// 绑定 Class 基于 id 为自己
container.bind(Weapon);

// 绑定 Class
container.bind('Service', Service);

// 获取Service 实例
const service = container.get('Service');

// 调用实例中的 print 方法
service.print();

```

以上代码简单的实现依赖注入与DI容器，可实现以下功能
* 每个Class都可以重新绑定
* 注入方式是构造函数（还有其他方式，这里就不一一实现了）
* 注入分为两种，1,基于 id 的注入 @Inject. 2, 基于类型的注入
* 循环注入（没有实现单例，也没处理循环依赖问题）

**思考**
- **反射只是提供了一种机制，让我们更方便的从Class内部渠道想要的数据，依赖注入也可以提供其他方式来实现。**
- **我们把所有的类的实例化交给了容器去做，被依赖放只需要实现依赖类对应的接口。**
- **所有的类被统一绑定与查找，这个操作不是在类文件内去操作，更易于管理（替换，修改，删除）。**
