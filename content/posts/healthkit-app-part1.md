---
title: "HealthKitアプリを作る（その１）"
date: 2017-12-21T17:43:55+09:00
tags: ["iOS", "Swift", "HealthKit"]
categories: ["N.Yamamoto"]
---

Apple Watch Series3を買いました。  
Series1からの買い替えです。  
CellularじゃなくてGPSの方です。  
<br />
Apple Watch Series3を使った感想は「去年買っておけばよかった」です。  
<br />
Nike+ Run Clubで走るの楽しい。  
とりあえず２週間続いてます。  
<br />
目標は「フルマラソン走ってた時の体重に戻して、来シーズンフルマラソン走る」ことにしました。  
<br />
「アクティビティ」と「Nike+ Run Club」のアプリでも記録は見られるのですが、  
ジョギングを習慣化するために、走ったログを簡単に参照できるアプリを作ってみます。  
ついでに体重の変化も見たいですね。（体重が減る前提）  
<br />
### 「HealthKitアプリを作る」不定期連載その１  
<br />
iOS11 / Swift4を使います。  
<br />
今回は、自分のiPhoneに登録してある体重、ワークアウトの記録をXcodeのコンソールで見るところまでです。  
<br />
**1.おもむろにXcodeを開いてシングルページアプリケーションのプロジェクトを作る**  
<br />
**2.プロジェクトでHealthKitが使えるようにする**  
<br />
TARGETS→Capabiliteis→HealthKitをONにする  
<br />
TARGETS→General→Linked Frameworks and Librariesに「HealthKit.framework」と「HealthKitUI.framework」を設定  
<br />
Info.plistを開いて"Add Row"して 「Privacy - Health Update Usage Description」を追加  
<br />
続けて"Add Row"して「Privacy - Health Share Usage Description」を追加  
<br />
**3.このアプリからiOS内のデータにアクセスする許可をもらう画面を出す**  
<br />
ViewController.swiftの viewDidLoadあたりをこんなかんじで

```
    let myHealthStore = HKHealthStore()

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        
        let readTypes = Set([
            HKWorkoutType.workoutType(),
            HKQuantityType.quantityType(forIdentifier: HKQuantityTypeIdentifier.bodyMass)!
        ])
        let writeTypes = Set([
            HKQuantityType.quantityType(forIdentifier: HKQuantityTypeIdentifier.bodyMass)!
        ])
        
        myHealthStore.requestAuthorization(toShare: writeTypes, read: readTypes, completion: { success, error in
            if success {
                print("Success")
            } else {
                print("Error")
            }
        })
    }
```

体重の書き込み許可  
ワークアウト、体重への読み込み許可  
をお願いしています  
<br />

**4.次に体重データを取得してみる**  
<br />
12月の1ヶ月間の体重の最大と最小値を取得。  
日付は直書きですが、1日ごとにループしてやれば体重の変化は取れそうですね。  

```
    func getWeight(){
        // 取得する期間を設定
        let dateformatter = DateFormatter()
        dateformatter.dateFormat = "yyyy/MM/dd"
        let startDate = dateformatter.date(from: "2017/12/01")
        let endDate = dateformatter.date(from: "2018/01/01")
        
        // 取得するデータを設定
        let typeOfWeight = HKObjectType.quantityType(forIdentifier: HKQuantityTypeIdentifier.bodyMass)
        let statsOptions: HKStatisticsOptions = [HKStatisticsOptions.discreteMin, HKStatisticsOptions.discreteMax]
        
        let predicate = HKQuery.predicateForSamples(withStart: startDate, end: endDate, options: HKQueryOptions.strictStartDate)
        let query = HKStatisticsQuery(quantityType: typeOfWeight!, quantitySamplePredicate: predicate, options: statsOptions, completionHandler: { (query, result, error) in
            if let e = error {
                print("Error: \(e.localizedDescription)")
                return
            }
            DispatchQueue.main.async {
                guard let r = result else {
                    return
                }
                let min = r.minimumQuantity()
                let max = r.maximumQuantity()
                if min != nil && max != nil {
                    print("\(r.startDate) : \(r.endDate) 最小:\(min!) 最大:\(max!)")
                }
            }
        })
        myHealthStore.execute(query)
    }

```

実機で実行したら動いたので次へ。  
<br />
**5.ワークアウトを取得してみる**  
<br />

```
    func getWorkout(){
        // 取得する期間を設定
        let dateformatter = DateFormatter()
        dateformatter.dateFormat = "yyyy/MM/dd"
        let startDate = dateformatter.date(from: "2017/12/01")
        let endDate = dateformatter.date(from: "2018/01/01")

        // 取得するデータを設定
        let predicate = HKQuery.predicateForSamples(withStart: startDate, end: endDate, options: [])
        let sort = [NSSortDescriptor(key: HKSampleSortIdentifierStartDate, ascending: true)]
        let q = HKSampleQuery(sampleType: HKObjectType.workoutType(), predicate: predicate, limit: 0, sortDescriptors: sort, resultsHandler:{
            (query, result, error) in

            if let e = error {
                print("Error: \(e.localizedDescription)")
                return
            }
            DispatchQueue.main.async {
                guard let r = result else {
                    return
                }
            
                let workouts = r as! [HKWorkout]
                for workout in workouts {
                    print(workout.startDate)
                    print(workout.totalDistance!)
                    print(workout.totalEnergyBurned!)
                }
            }
        })
        
        myHealthStore.execute(q)
    }
```

こちらはワークアウトの回数分データが取れました。  
めでたしめでたし。  
<br />
この後はデータをどう表示するかなのですが。  
まだ決めてないのでそのうち。  
<br />
<br />


