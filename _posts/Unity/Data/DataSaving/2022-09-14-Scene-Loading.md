---
title: Scene Loading
author: unwoo52
date: 2022-09-14 00:00:00 +09:00
categories: [Unity, UI]
tags: [Unity, UI]
---

## Scene Load

![imagename](/assets/image/Unity/Data/SceneLoading/001.png)

<details>
<summary>SSceneLoaderScript</summary>
<div markdown="1">

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class SSceneLoaderScript : MonoBehaviour
{
    public static SSceneLoaderScript instance = null;
    public Slider Sliderslider;
    private void Awake()
    {
        instance = this;
        DontDestroyOnLoad(this);
    }
    public void SceneChange(int i)
    {
        StartCoroutine(Loading(i));
    }

    IEnumerator Loading(int i)
    {
        yield return UnityEngine.SceneManagement.SceneManager.LoadSceneAsync("SLoad");

        Sliderslider.gameObject.SetActive(true);
        Sliderslider.value= 0;

        StartCoroutine(LoadingTarget(i));
    }

    IEnumerator LoadingTarget(int i)
    {
        AsyncOperation asyncOperation = UnityEngine.SceneManagement.SceneManager.LoadSceneAsync(i);
        asyncOperation.allowSceneActivation= false;
        while (!asyncOperation.isDone)
        {
            Sliderslider.value = asyncOperation.progress / 0.9f;
            if(asyncOperation.progress >= 0.9f)
            {
                asyncOperation.allowSceneActivation = true;
            }
            yield return null;
        }
        yield return new WaitForSeconds(1);
        Sliderslider.gameObject.SetActive(false);
       
    }
}
```

</div>
</details>


<details>
<summary>STitle</summary>
<div markdown="1">

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class STitle : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        if(Input.GetKeyDown(KeyCode.Space))
        {
            //Application.LoadLevel(5); << 여러 문제를 내포하고 있기 때문에 보통 사용되지 않음
            //UnityEngine.SceneManagement.SceneManager.LoadSceneAsync(5);

            SSceneLoaderScript.instance.SceneChange(5);
        }
    }
}
```

</div>
</details>
