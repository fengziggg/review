btn  
slider  
dropdown  
toggle  
scrollRect(scrollbar):  
- 一般还在content添加Layout组件更改布局  
- content添加size filter更改content的size  


```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class p7 : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        p1();
        p2();
        p3();
        p4();
        p5();
        p6();
    }

    // Update is called once per frame
    void Update()
    {
        
    }

    private void p6(){
        TMP_Dropdown dd = GameObject.Find("Dropdown").GetComponent<TMP_Dropdown>();
        print(dd.value);
        dd.onValueChanged.AddListener((int index)=>{
            int new_idx = dd.options.Count;
            print($"select idx: {index}, new index: {new_idx}");
            dd.options.Add(new TMP_Dropdown.OptionData("test"+new_idx));
        });
    }

    private void p5(){
        GameObject prefab = Resources.Load<GameObject>("Item");
        GameObject go = GameObject.Find("ScrollView");
        ScrollRect sc = go.GetComponent<ScrollRect>();
        sc.vertical = true;
        sc.horizontal = false;
        print(prefab);
        print(sc.content);
        for(int i=0;i<20;i++){

            GameObject item = Instantiate(prefab, sc.content);
            // float y = -(prefab.transform as RectTransform).rect.height*i;
            // Vector3 pos = new Vector3(0, y, 0);
            // item.transform.localPosition = pos;
            item.GetComponentInChildren<TMP_Text>().text = "item" + i;
        }
        sc.onValueChanged.AddListener((Vector2 pos)=>{print($"value changed: {pos}");});
        // StartCoroutine(p5_1(sc));
    }

    
        private IEnumerator p5_1(ScrollRect sc){
            while(true){
                print(sc.normalizedPosition);
                print(sc.content.sizeDelta);
                yield return new WaitForSeconds(2);
            }
        }

    private void p4(){
        GameObject go = GameObject.Find("Slider");
        Slider sl = go.GetComponent<Slider>();
        sl.onValueChanged.AddListener(delegate(float value){
            print($"{sl.name} change value {value}");
        });
    }

    private void p3(){
        GameObject go = GameObject.Find("InputField(TMP)");
        TMP_InputField inputF = go.GetComponent<TMP_InputField>();
        inputF.onSelect.AddListener((string msg)=>{
            print($"select {inputF.name}: " + msg);
        });
        inputF.onDeselect.AddListener((string msg)=>{
            print($"cancel select {inputF.name}: " + msg);
        });
        inputF.onValueChanged.AddListener((string msg)=>{
            print($"change value {inputF.name}: " + msg);
        });
    }

    private void p2(){
        GameObject go = GameObject.Find("group");
        ToggleGroup group = go.GetComponent<ToggleGroup>();
        group.allowSwitchOff = false;
        Toggle [] tgs = go.transform.GetComponentsInChildren<Toggle>();
        tgs[1].onValueChanged.AddListener((bool isOn)=>p2_1(tgs[1], isOn));
        tgs[0].onValueChanged.AddListener(delegate(bool isOn){
            print($"{tgs[0].name} change status: {isOn}");
        });
    }

    private void p2_1(Toggle tg, bool isOn){
        print($"{tg.name} change status {isOn}");
    }

    public void p2_2(bool isON){
        print("toggle editor binding" + isON);
    }

    private void p1(){
        GameObject go = GameObject.Find("Button");
        Button btn = go.GetComponent<Button>();
        btn.onClick.AddListener(()=>{print("Add btn callbck1111");});
        btn.onClick.AddListener(()=>p1_2(btn));

    }

    public void p1_1(){
        print("Add btn callback2:Editor binding");
    }

    public void p1_2(Button btn){
        btn.onClick.RemoveAllListeners();
    }
}

```
