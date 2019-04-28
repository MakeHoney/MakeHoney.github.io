---
title: "ListView와 ViewHolder Pattern"
layout: post
date: 2019-04-28 19:20
headerImage: false
tag:
- ListView
- ViewHolder
- Kotlin
- Android
category: android
author: Byunghun Lee
star: false
description: ListView Summary
---


### Prerequisites

> ListView는 현재 RecyclerView로 대체되어 deprecated된 뷰 컨테이너(뷰 그룹)입니다. 하지만 ViewHolder pattern의 내부 원리를 이해하기 위해서는 ListView를 사용하여 CustomAdapter를 구현하는 게 좋을 것이라 판단하여 포스팅하게 됐습니다.

예제 실행 화면입니다. 이해를 돕기 위해서 첨부하겠습니다.
![1556460781713](https://user-images.githubusercontent.com/48861849/56865660-76184f80-6a0b-11e9-8f27-df24c7c564e7.jpg)



### ListView

ListView는 여러개의 View들을 담는 뷰 컨테이너입니다. 

![1556462318133](https://user-images.githubusercontent.com/48861849/56865953-dc52a180-6a0e-11e9-87fa-7c914a49e605.jpg)

위 그림에서 보이듯, 스크린 사이즈에는 제약이 있기에 ListView에 존재하는 모든 항목을 보여주지는 못합니다. ListView는 내부적으로 스크린에 보여지는 View들만을 생성하고 스크롤링될 때마다 어댑터의 getView 메소드를 통해 새로운 뷰를 불러옵니다.

**\* 예제에서는 여러개의 Layout View들을 ListView에 담습니다. Layout View는 위 그림에서 하나의 View를 의미합니다. 예제에서 Layout View는 ImageView와 두 개의 TextView로 이루어져 있습니다. Layout View도 View이기에 의미에 혼선이 있을 수 있으므로, 앞으로 Layout View를 Layout 객체라고 표현하겠습니다.**

- 스크롤 시 ListView는 getView 메소드를 통해서 어댑터에게 Layout 객체를 요청합니다. 

- **요청을 받은 어댑터는 두 번째 인자로 전달 받은 convertView를 가지고 해당 작업을 수행합니다.** 

- **convertView가 null이라면 inflate를 통해서 Layout 객체를 얻어온 뒤,**

- **해당 Layout에 존재하는 View들의 객체(TextView, ImageView 등)를 각각 얻어서 해당 View들의 내용을 채워(set)줍니다.**

굵은 글씨로 된 작업이 어댑터의 getView가 하는 작업들이고 이를 그림으로 표현하면 아래와 같습니다.

![1556460788238](https://user-images.githubusercontent.com/48861849/56865708-d1e2d880-6a0b-11e9-8157-bd2e902b6ee6.jpg)

제가 짠 어댑터의 코드는 아래와 같습니다.

**CustomAdapter.kt**
```kotlin
class CustomAdapter (private val ctx: Context) : BaseAdapter() {

    override fun getCount(): Int = initData().size

    override fun getItem(p0: Int): Any? = null

    override fun getItemId(p0: Int): Long = 0
    
    override fun getView(position: Int, convertView: View?, parent: ViewGroup?): View? {
        var view = convertView
        if (view == null) {
            view = LayoutInflater.from(ctx).inflate(R.layout.row, null)
        }
        var flag = view?.findViewById<ImageView>(R.id.imageView)
        var nation = view?.findViewById<TextView>(R.id.textView2)
        var capital = view?.findViewById<TextView>(R.id.textView3)

        val dataArray = initData()
        dataArray[position].let {
            flag?.setImageResource(it["flag"] as Int)
            nation?.text = it["nation"] as String
            capital?.text = it["capital"] as String
        }

        return view
    }

    // initData는 단순히 데이터를 초기화를 위한 메소드입니다.
    private fun initData(): ArrayList<HashMap<String, Any>> {
        val flags = intArrayOf (
            R.drawable.imgflag1,
            R.drawable.imgflag2,
            R.drawable.imgflag3,
            R.drawable.imgflag4,
            R.drawable.imgflag5,
            R.drawable.imgflag6,
            R.drawable.imgflag7,
            R.drawable.imgflag8
        )

        val nations = arrayOf("토고", "프랑스", "스위스", "스페인", "일본", "독일", "브라질", "대한민국")
        val capitals = arrayOf("로메", "파리", "베른", "마드리드", "도쿄", "베를린", "브라질시티", "서울")

        val list = ArrayList<HashMap<String, Any>>()

        val len = flags.size
        var i = 0
        while (i < len) {
            val map = HashMap<String, Any>()
            map["flag"] = flags[i]
            map["nation"] = nations[i]
            map["capital"] = capitals[i]
            list.add(map)
            i++
        }
        return list
    }
}
```

- _InitData 메소드는 데이터 베이스를 대신하여 목업 데이터를 제공해주는 메소드라고 생각하시면 됩니다._

- _getView 메소드의 position 파라미터를 인덱스로 사용하여 데이터에 접근합니다._

  

**MainActivity.kt**
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val adapter = CustomAdapter(this)
        listView.adapter = adapter
    }
}
```



### 문제점

위 코드(getView)는 크게 두 가지 문제점을 가지고 있습니다.

1. 매 getView 요청마다 불필요한 **inflate**가 일어날 수 있다.

2. 매 getView 요청마다 불필요하게 **findViewById**를 한다.

위 두가지 연산(inflate, findViewById)은 꽤 큰 비용을 수반하는 작업입니다. 

따라서 스크롤 시 매끄럽지 못한 사용자 경험을 제공할 수 밖에 없습니다.

- 1번 문제점의 경우 사실 convertView의 null 예외 처리를 통해서 처리를 한 상태입니다. 하지만 이렇게 직접 예외 처리를 해주지 않는 경우에 하나의 Layout 객체를 불러올 때마다 inflate가 일어나게됩니다.

- ViewHolder pattern은 2번 문제점을 개선하기 위해 사용하고, 2009 Google I/O에서 발표된 것으로 알고 있습니다.



### ListView with 'ViewHolder'

- ViewHolder는 각 Layout 객체에 존재하는 View 객체들을 말 그대로 hold하(잡아두)는 역할을 합니다.

- ViewHolder는 어댑터에서 생성되며 각 Layout 객체에 tag를 통해서 바인딩됩니다.

그림으로 표현하면 아래와 같은 모양이 됩니다.

![1556462319356](https://user-images.githubusercontent.com/48861849/56865958-eeccdb00-6a0e-11e9-9ec4-067078031655.jpg)


이를 코드로 표현하면 아래와 같습니다.

**CustomAdapter.kt**
```kotlin
class CustomAdapter (private val ctx: Context) : BaseAdapter() {
    data class ViewHolder(var flag: ImageView?, var nation: TextView?, var capital: TextView?)

. . .
. . .
. . .

    override fun getView(position: Int, convertView: View?, parent: ViewGroup?): View? {
        var view = convertView
        if (view == null) {
            view = LayoutInflater.from(ctx).inflate(R.layout.row, null)
            val flag = view?.findViewById<ImageView>(R.id.imageView)
            val nation = view?.findViewById<TextView>(R.id.textView2)
            val capital = view?.findViewById<TextView>(R.id.textView3)

            val dataArray = initData()
            dataArray[position].let {
                flag?.setImageResource(it["flag"] as Int)
                nation?.text = it["nation"] as String
                capital?.text = it["capital"] as String
            }

            val holder = ViewHolder(flag, nation, capital)
            view.tag = holder // layout 객체에 holder를 바인딩
        } else {
            val holder = view.tag as ViewHolder
            val dataArray = initData()
            dataArray[position].let {
                with(holder) {
                    flag?.setImageResource(it["flag"] as Int)
                    nation?.text = it["nation"] as String
                    capital?.text = it["capital"] as String
                }
            }
        }
        return view
    }

. . .
. . .
. . .
```

- _**convertView가 null인 경우** inflate를 통해서 Layout 객체를 얻어오고 Layout 객체의 findViewById 메소드를 통해서 내부 View 객체들을 얻어오는 연산을 수행합니다._

- _Layout 내부 View 객체들은 ViewHolder에 저장(saving)되고 ViewHolder는 Layout 객체에 tag를 통해 바인딩(binding or caching)됩니다._

- _**convertView가 null이 아닌 경우** tag를 통해서 holder를 불러오고 findViewById 연산없이 바인딩(or caching)된 데이터를 바로 사용합니다._

- _ViewHolder Pattern은 이같은 방식을 통해 완전한 Layout 객체 **재활용**을 구현해냅니다._