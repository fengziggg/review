```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class p6 : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        Transform imgTrans = this.transform.Find("Image");
        Transform rawTrans = this.transform.Find("RawImage");
        Transform textTrans = this.transform.Find("Text");

        Image img = imgTrans.GetComponent<Image>();
        img.sprite = Resources.Load<Sprite>("3DModels/ORANGE_BODY_SKULL");
        (imgTrans as RectTransform).sizeDelta = new Vector2(100, 100);

        RawImage rawImg = rawTrans.GetComponent<RawImage>();
        rawImg.uvRect = new Rect(0, 0, 0.5f, 0.5f);

        textTrans.GetComponent<Text>().text = "hellow world";

    }

    // Update is called once per frame
    void Update()
    {
        
    }
}

```
