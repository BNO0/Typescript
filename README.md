- [Interface type Union](#interface-type-union)
- [in is as - Type Guard(타입가드), Type Assertion(타입 단언)](#in-is-as)
- [Utility Type](#utility-type)
	- [Partial](#partial)
	- [Pick](#pick)
	- [Omit](#omit)
	- [Exclude](#exclude)
	- [Required](#required)
	- [Record](#record)
	- [ReturnType](#returntype)
- [Mapped Types](#mapped-types)
- [조건부(?) 타입, infer](#conditional-type-infer)
- [다른 type의 속성type 재사용 - Lookup Types](#lookup-types)
- [타입이 여러개일때 특정 타입의 메소드 사용 - satisfies](#satisfies)
- [함수타입을 재사용 - 호출 시그니처](#call-signature)
- [속성이 가변적일 때 - 인덱스 시그니처](#index-signature)
- [Overloading](#overloading)
- [Generics](#generics)

# Interface type Union
- Interface : implements 가능, extends 를 통해 확장, 선언병합 가능
- Type : implements 가능, &를 통해 확장, 선언병합 불가능
- Union : 여러개의 타입이 가능한 새로운 타입, 항상 type으로 선언해야함
```typescript
interface Animal{					// Interface
	name: string;
}
//interface Animal{					// 선언병합 가능
//	honey: boolean;
//}
//const temp: Animal = {				// Interface Animal 선언 병합
//	name: 'bear',
//	honey: true,
//}
interface Bear extends Animal {
	honey: boolean;
}
const temp: Bear = {
	name: 'bear',
	honey: true,
}

type AnimalType = {				// Type
	name2: string;
}
type BearType = AnimalType & {
	honey2: boolean;
}
const tempAnimal: BearType = {
	name2: 'bear',
	honey2: true,
}
type NewType = Animal | Bear | AnimalType | BearType;	//Union
const unionBear: NewType = {
	name: 'bear',
	name2: 'bear2',
	honey: true,
	honey2: false,
}
```

# in is as
- `[객체] in [프로퍼티]` - Type Guard(타입 가드)
	- 런타임에 변수나 객체의 타입을 안전하게 확인
	- 객체가 프로퍼티를 가지고 있는지 판단
- `function [함수명] ([매개변수]: any): [매개변수] is [type] { return [boolean]}` - Type Guard(타입 가드)
	- true 반환 시 함수가 호출된 범위 내에서는 매개변수를 type 으로 판단
- `[객체] as [type]` - Type Assertion(타입 단언)
	- 개발자가 변수나 객체를 특정 타입이라고 주장하는 것
	- 객체를 type으로 판단
```typescript
interface UserType {
	id: number;
	text: string;
}
const isUserType = (object: any): object is UserType => {
	return (
		"id" in object &&				// object.id 존재여부 판단
		"text" in object &&				// object.text 존재여부 판단
		typeof (object as UserType).id === "number" &&	// object를 UserType으로 판단하고 object.text의 id 가 number 인지 판단
		typeof (object as UserType).text === "string"	// object를 UserType으로 판단하고 object.text의 type 이 string 인지 판단
	);	//위 조건을 모두 충족하면 isUserType이 호출된 범위 내에서 object 를 UserType으로 판단
};
```

# Utility Type
```typescript
interface Address {
	id: string;
	email: string;
	address: string;
}
```
## Partial
- `const [변수명]: Partial<[Interface / Type]>` - Interface / Type 의 일부속성만 가지는 것을 가능하게함
```typescript
const partialTest: Partial<Address> = {};
const partialTest2: Partial<Address> = { email: 'hone@abc.com'};
```

## Pick
- `type [type명]: Pick<[Interface / Type], "[프로퍼티1]" | "[프로퍼티2]" | ...>` - Interface / Type 의 일부속성만 가지는 타입 정의
```typescript
type pickType = Pick<Address, 'id' | 'email'>;	// Address의 id, email 프로퍼티만 가지는 새로운 타입 정의
const pickTest: pickType = {
	id: 'abc',
	email: 'abc@google.com',
}
```

## Omit
- `type [type명]: Omit<[Interface / Type], "[프로퍼티1]" | "[프로퍼티2]" | ...>` - Interface / Type 의 일부속성만 제거한 타입 정의
```typescript
type omitType = Omit<Address, 'id'>;		// Address의 id 프로퍼티를 제거한 새로운 타입 정의
const omitTest: omitType = {
	email: 'tt@abc@com',
	address: 'asdf',
}
```

## Exclude
- `let [변수명]: Exclude<[Union], "[type]" | "[type]" | ...>` - Union 의 일부타입을 제거한 타입으로 변수생성
```typescript
type excludeTestType = string | number;
let excludeTest: Exclude<excludeTestType, number> = 'abc';	// excludeTestType의 number type을 제거한 타입으로 변수할당
```

## Required
- `let [변수명]: Required<[Interface / Type]>` - Interface / Type 의 속성이 선택사항이라도 모든 속성이 있어야 변수할당가능
```typescript
type RequiredTestType = {
	abc: string,
	xyz?: string,	//선택사항
}
let temp: RequiredTestType = {
	abc: 'jone',	// xyz 없어도 됨
}
let requiredTest: Required<RequiredTestType> = {
	abc: 'jone',
	xyz: 'asdf',	// 없으면 error
}
```

## Record
- `let [변수명]: Record<[keyType], [Interface / Type]>` - 키가 keyType이고 값이 Interface / Type인 객체
```typescript
type recordType = 't1' | 't2' | 't3'
const recordTest: Record<recordType, Address> = {
	t1: {id: '1'; email: 'a@abc.com', address: 'zxcv'},
	t2: {id: '2'; email: 'b@abc.com', address: 'zxc'},
	t3: {id: '3'; email: 'c@abc.com', address: 'zx'},
}
```

## ReturnType
- `type [type명] = ReturnType<[함수]> = [value]` - 함수의 반환타입으로 구성된 타입
- `let [변수명]: ReturnType<typeof [함수명]> = [value]` - 함수의 반환타입으로 구성된 변수할당
```typescript
type returnType = ReturnType<()=> string>;	// string
type returnType2 = ReturnType<(s: string)=> void>;// void

function returnTypeFunction(str: string){
	return str;
}
const returnTypeTest: ReturnType<typeof returnTypeFunction> = 'hello';	//string
//const returnTypeTest: ReturnType<typeof returnTypeFunction> = true;	//error
```

# Mapped Types
- `[K in [type명]]: [newType]` - type명의 속성명을 가져와 newType으로 새로운 Type 생성
- 암시적으로 관계있는 타입을 묶어줄 때 사용 ( ex) 각 속성을 변경할 권한 체크)
```typescript
type MappedTypes = {[K in Address]: boolean};	// 반드시 K 포함
type MappedTypes2 = {[K in Address]?: string};	// 선택적으로 K 포함
const testMapped: MappedTypes = {
	id: true;
	email: false;
	address: true;
}
//const testMapped2: MappedTypes={}	// error
const testMapped2: MappedTypes2 = {};
```

# Conditional Type infer
- `type [type명] = [type1] extends [type2] ? [type3] : [type4]` - type1이 type2에 할당가능하다면 type3, 불가능하다면 type4로 type정의
- `infer [type]` - type으로 추론함
```typescript
type ConditionalType<T> = T extends string ? string : never;
const test1: ConditionalType<string> = 'string';		// string이 string에 할당가능하기 때문에 string으로 type 정의

type ConditionalType2<T> = T extends string | number ? T : never;
type test2 = ConditionalType2<string | number | boolean>;	// boolean 은 string | number 에 할당 불가능하기 때문에 test2: string | number

type ConditionalType3<T> = [T] extends [string | number] ? T : never;
type test3 = ConditionalType2<string | number | boolean>;	// [string | number | boolean] 은 [string | number] 에 할당 불가능하기 때문에 test3: never

type ConditionalType4<T> = T extends infer R ? R : null;
const testInfer: ConditionalType4<number> = 1;		// R이 number로 추론됨 > testInfer: number

type ConditionalType5<T> = T extends {a: infer A; b: 1} ? A : any;
type testInfer2 = ConditionalType5<{a: 'hi'; b: 1}>;		// A가 'hi'로 추론됨 > testInfer2: 'hi'

type ConditionalType6<T> = T extends {a: infer A; b: infer B} ? A & B : any;
type testInfer3 = ConditionalType5<{a: {tA: 1}; b: {tB: 2}}>;	// A가 {tA:1}, B가 {tB:2} 로 추론됨 > testInfer2: {tA:1} & {tB:2}

// infer 사용시 반환타입에 하드코딩 할 필요 없음
type InferExample<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
type TestInferExample1 = InferExample<()=> string>;	// TestInferExample1: string
type TestInferExample2 = InferExample<()=> number>;	// TestInferExample2: number
// const hardCoding: string | number = function();
// const inferValue: TestInferExample1 = function();
```

# Lookup Types
- `[newType]: [type | interface][[oldtype]]` - 다른 type의 속성type 재사용
```typescript
interface Car{
	id: number;
	name: string;
	brand: {
		popularity: number;
		logo: string;
		history: number;
	}
}
function updateCarBrand(id: Car['id'], newBrand: Car['brand']){	// Car의 id, Car의 brand 속성 type 사용
	...
}
```

# satisfies
- `const [변수명] = {} satisfies [type]` - 여러 type을 가지는 경우 특정 type에서 사용되는 메소드 사용가능, 컴파일 이전에 error체크가 가능
```typescript
type Colors = 'red' | 'green' | 'blue';
type RGB = [red: number, green: number, blue: number];
const palette = {
	red: [255,0,0],
	green: '#00ff00',
	blue: [0,0,255],
} satisfies Record<Colors, string | RGB>			// string일때 string method 호출 가능
const greenNormalized = palette.green.toUpperCase();	// satisfies가 없는경우 RGB일수도 있기 때문에 호출 불가능
```

# Call Signature
- 호출 시그니처
- `interface [함수명]{ ([매개변수]: [type]): [returnType] }` - 함수타입을 재사용하고 싶을 때 Interface생성

# Index Signature
- 인덱스 시그니처
- `[[키]: [type]: unknown]` - 모든속성을 알지 못할 때 (속성을 추가하는 경우)
```typescript
interface getLikeNumber {		// 호출 시그니처
	(like: number): number;
}
interface Post {
	[key: string]: unknown;	// 인덱스 시그니처
	id: number;
	title: string;
	getLikeNumber: getLikeNumber;
}
const post1: Post = {
	id: 1,
	title: 'post1',
	getLikeNumber(like: number) {
		return like
	}
}
post1['description'] = 'description1';	// [key: string]: unknown

interface Names {
	[item: number]: string;
}
const userNames: Names = ['John', 'Kim', 'Joe'];
console.log(userNames[0], userNames[1], userNames[2]);	// John, Kim, Joe
```

# Overloading
```typescript
function add(a: string, b: string): string;
function add(a: number, b: number): number;
function add(a: any, b: any): any{
	return a+b;
}
```

# Generics
```typescript
function makeArr<T, Y>(x: X, y: Y): [X, Y]{
	return [x, y];
}
const array1 = makeArr<number, number>(4,5);
const array2 = makeArr<string, string>('a','b');

console.log(array1);	// [4,5]
console.log(array2);	// ['a', 'b']


interface Vehicle<T>{
	name: string;
	color: string;
	option: T;
}
const car: Vehicle<{price: number}>= {
	name: 'Car',
	color: 'red',
	option: {
		price: 1000
	}
}
const bike: Vehicle<boolean> = {
	name: 'Bike',
	color: 'green',
	option: true,
}


const makeFullName = <T extends { firstName: string, lastName: string }>(obj: T) => {
	return {
		...obj,
		fullName: obj.firstName + ' ' + obj.lastName
	}
}
console.log(makeFullName({firstName: 'Jone', lastName: 'Doe', location: 'Seoul'}););	// {firstName: 'Jone', lastName: 'Doe', location: 'Seoul', fullName: 'John Doe'}
```