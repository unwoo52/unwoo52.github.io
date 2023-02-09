---
title: Transform과 Component
author: unwoo52
date: 2022-12-12 00:00:00 +09:00
categories: [Win32API]
tags: [Win32API]
---

## Component Class 

<details>
<summary markdown="span"> 
Component.h 전체 코드
</summary>

<div markdown="1">

```cpp
#pragma once
#include <DirectXMath.h>
using namespace DirectX;

class GameObject;

class Component
{
protected:	
public:
	GameObject* gameObject = nullptr;
	virtual ~Component() {};
	virtual void Update() = 0;
};
```

</div>

</details>



## GameObject Class

<details>
<summary markdown="span"> 
GameObject.h 전체 코드
</summary>

<div markdown="1">

```cpp
#pragma once
#include "../Component/ComponentTypes.h"
#include <list>
#include <vector>
#include <memory>
using namespace std;

class GameObject
{
	list<shared_ptr<Component>> componentList;
public:
	GameObject();
	Transform* GetTransform();
	void AddComponent(shared_ptr<Component> com);
	template<class T>
	T* GetComponent()
	{
		T* ptr = nullptr;
		for (shared_ptr<Component> comp : componentList)
		{			
			ptr = dynamic_cast<T*>(comp.get());
			if (ptr != nullptr)
			{
				break;
			}
		}
		return ptr;
	}
};
```

</div>

</details>



컴포넌트 리스트를 갖고 있다. \[!!여기서 컴포넌트 리스트의 첫번째는 반드시 transform 컴포넌트이다.\]

- GetComponent 메소드

```cpp
T* GetComponent()
	{
		T* ptr = nullptr;
		for (shared_ptr<Component> comp : componentList)
		{			
			ptr = dynamic_cast<T*>(comp.get());
			if (ptr != nullptr)
			{
				break;
			}
		}
		return ptr;
	}
```

GetComponent 메소드를 잠시 살펴보자.

찾고자 하는 컴포넌트종류의 템플릿(T)를 받아 포인터를 선언한 후, for문을 통해 컴포넌트 리스트들을 포인터에 형변환을 시도해본다. 만약 성공한다면 포인터를 리턴하고 for문을 종료.

-----
<details>
<summary markdown="span"> 
GameObject.cpp 전체 코드
</summary>

<div markdown="1">

```cpp
#include "GameObject.h"

GameObject::GameObject()
{
	shared_ptr<Transform> ptr = make_shared<Transform>();
	AddComponent(ptr);
}

Transform* GameObject::GetTransform()
{	
	return dynamic_cast<Transform*>(componentList.begin()->get());
}

void GameObject::AddComponent(shared_ptr<Component> com)
{	
	com->gameObject = this;
	componentList.push_back(com);
}
```

</div>

</details>

- GetTransform 메소드

```cpp
Transform* GameObject::GetTransform()
{	
	return dynamic_cast<Transform*>(componentList.begin()->get());
}
```

위에서 컴포넌트리스트의 가장 첫번째는 트랜스폼이라고 말했었다.

componentList.begin()->get()로 받아온 트랜스폼은 컴포넌트포인터 형식이다. 이를 dynamic_cast를 통해 트랜스폼 포인터로 형변환 후 리턴.


## Transfrom

카메라 클래스에 있던 행렬 연산들을 옮김 32분

프로젝션 뷰 는 카메라에 그대로 있어야 함. 월드 및 포지션만 옮김34

트랜스폼에 함수들 정의 ~43

필드 : 위치값 등 추가 44

58분에 각 함수에 셋월드매트릭스()를 추가함 << 이 함수 가서 어떤 함순지 다시 확인해보자

1:00 업데이트월드매트릭스() 만들기, 좀 어렵다 검색해보자 [ㅇ](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=bastard9&logNo=140182012627)


1:07 컴포넌트클래스에 순수가상함수 업데이트() 만듬

## Renderer Class

<details>
<summary markdown="span"> 
Renderer.h 전체 코드
</summary>

<div markdown="1">

```cpp
#include "../Buffers.h"
#include "../ConstantBufferTypes.h"
#include "Component.h"
#include "../Object/Mesh.h"

class Renderer : public Component
{
	ID3D11Device* device = nullptr;
	ID3D11DeviceContext* dc = nullptr;
	ConstantBuffer<CB_VS_Transform>* cb_transform = nullptr;
	ID3D11ShaderResourceView* texture = nullptr;
	VertexBuffer<Vertex> vb;
	IndexBuffer ib;
	//for Assimp
	vector<Mesh> meshes;
	bool LoadFile(const string& filePath);
	void ProcessNode(aiNode* node, const aiScene* scene);
	Mesh ProcessMesh(aiMesh* mesh, const aiScene* scene);	
public:
	void Update() override;
	bool Initialize(ID3D11Device* device, ID3D11DeviceContext* dc, ID3D11ShaderResourceView* tex, ConstantBuffer<CB_VS_Transform>& cb_vs_transform);
	bool Initialize(const string& filePath, ID3D11Device* device, ID3D11DeviceContext* dc, ID3D11ShaderResourceView* tex, ConstantBuffer<CB_VS_Transform>& cb_vs_transform);
	void SetTexture(ID3D11ShaderResourceView* tex);
	void Draw(const XMMATRIX vpMat);	
};
```

- Renderer::LoadFile

파일을 로드

- Renderer::ProcessNode

- Renderer::ProcessMesh

</div>

</details>

<details>
<summary markdown="span"> 
Renderer.cpp 전체 코드
</summary>

<div markdown="1">

```cpp
#include "Renderer.h"
#include "../../../ComException.h"
#include "../Object/GameObject.h"
bool Renderer::LoadFile(const string& filePath)
{
    Assimp::Importer importer;
    const aiScene* pScene = importer.ReadFile(filePath, aiProcess_Triangulate | aiProcess_ConvertToLeftHanded);
    if (pScene == nullptr) return false;
    ProcessNode(pScene->mRootNode, pScene);
    return true;
}
void Renderer::ProcessNode(aiNode* node, const aiScene* scene)
{
    for (UINT i = 0; i < node->mNumMeshes; ++i)
    {
        aiMesh* mesh = scene->mMeshes[node->mMeshes[i]];
        meshes.push_back(this->ProcessMesh(mesh, scene));
    }

    for (UINT i = 0; i < node->mNumChildren; ++i)
    {
        ProcessNode(node->mChildren[i], scene);
    }
}
Mesh Renderer::ProcessMesh(aiMesh* mesh, const aiScene* scene)
{
    vector<Vertex> vertices;
    vector<DWORD> indices;

    for (UINT i = 0; i < mesh->mNumVertices; ++i)
    {
        Vertex v;
        v.pos = { mesh->mVertices[i].x, mesh->mVertices[i].y, mesh->mVertices[i].z };
        if (mesh->mTextureCoords[0])
        {            
            v.uv.x = (float)mesh->mTextureCoords[0][i].x;
            v.uv.y = (float)mesh->mTextureCoords[0][i].y;
        }
        vertices.push_back(v);
    }

    for (UINT i = 0; i < mesh->mNumFaces; ++i)
    {
        aiFace face = mesh->mFaces[i];
        for (UINT n = 0; n < face.mNumIndices; ++n)
        {
            indices.push_back(face.mIndices[n]);
        }
    }

    return Mesh(device, dc, vertices, indices);
}
void Renderer::Update()
{
}

bool Renderer::Initialize(ID3D11Device* device, ID3D11DeviceContext* dc, ID3D11ShaderResourceView* tex, ConstantBuffer<CB_VS_Transform>& cb_vs_transform)
{
    this->device = device;
	this->dc = dc;
	texture = tex;
	cb_transform = &cb_vs_transform;
    try
    {
        Vertex v[] =
        {
            //Front
            Vertex(-0.5f, 0.5f, -0.5f, XMFLOAT3(1,1,1),{0,0}),//0
            Vertex(0.5f, 0.5f, -0.5f, XMFLOAT3(1,1,1), {1,0}),//1
            Vertex(-0.5f, -0.5f, -0.5f, XMFLOAT3(1,1,1), {0,1}),//2        
            Vertex(0.5f, -0.5f, -0.5f, XMFLOAT3(1,1,1), {1,1}),//3        
            //Back
            Vertex(-0.5f, 0.5f, 0.5f, XMFLOAT3(1,1,1),{1,0}),//4
            Vertex(0.5f, 0.5f, 0.5f, XMFLOAT3(1,1,1), {0,0}),//5
            Vertex(-0.5f, -0.5f, 0.5f, XMFLOAT3(1,1,1), {1,1}),//6        
            Vertex(0.5f, -0.5f, 0.5f, XMFLOAT3(1,1,1), {0,1}),//7 
            //Up
            Vertex(-0.5f, 0.5f, 0.5f, XMFLOAT3(1,1,1),{0,0}),//8
            Vertex(0.5f, 0.5f, 0.5f, XMFLOAT3(1,1,1), {1,0}),//9
            Vertex(-0.5f, 0.5f, -0.5f, XMFLOAT3(1,1,1), {0,1}),//10        
            Vertex(0.5f, 0.5f, -0.5f, XMFLOAT3(1,1,1), {1,1}),//11 
            //Down
            Vertex(-0.5f, -0.5f, 0.5f, XMFLOAT3(1,1,1),{1,0}),//12
            Vertex(0.5f, -0.5f, 0.5f, XMFLOAT3(1,1,1), {0,0}),//13
            Vertex(-0.5f, -0.5f, -0.5f, XMFLOAT3(1,1,1), {1,1}),//14        
            Vertex(0.5f, -0.5f, -0.5f, XMFLOAT3(1,1,1), {0,1}),//15 
            //Right
            Vertex(0.5f, 0.5f, -0.5f, XMFLOAT3(1,1,1),{0,0}),//16
            Vertex(0.5f, 0.5f, 0.5f, XMFLOAT3(1,1,1), {1,0}),//17
            Vertex(0.5f, -0.5f, -0.5f, XMFLOAT3(1,1,1), {0,1}),//18        
            Vertex(0.5f, -0.5f, 0.5f, XMFLOAT3(1,1,1), {1,1}),//19 
            //Left
            Vertex(-0.5f, 0.5f, -0.5f, XMFLOAT3(1,1,1),{0,0}),//20
            Vertex(-0.5f, 0.5f, 0.5f, XMFLOAT3(1,1,1), {1,0}),//21
            Vertex(-0.5f, -0.5f, -0.5f, XMFLOAT3(1,1,1), {0,1}),//22        
            Vertex(-0.5f, -0.5f, 0.5f, XMFLOAT3(1,1,1), {1,1}),//23 

        };

        HRESULT hr = vb.Initialize(device, v, ARRAYSIZE(v));
        COM_ERROR_IF_FAILED(hr, L"버텍스 버퍼 생성에 실패 하였습니다.");

        DWORD indices[] =
        {
            //Front
            0, 1, 2,
            1, 3, 2,
            //Back
            4, 6, 5,
            5, 6, 7,
            //Up
            8, 9, 10,
            9, 11, 10,
            //Down
            12, 14, 13,
            13, 14, 15,
            //Right
            16, 17, 18,
            17, 19, 18,
            //Left
            20, 22, 21,
            21, 22, 23
        };

        hr = ib.Initialize(device, indices, ARRAYSIZE(indices));
        COM_ERROR_IF_FAILED(hr, L"인덱스 버퍼 생성에 실패 하였습니다.");
    }
    catch (ComException& ex)
    {
        Debug::Log(ex.what());
        return false;
    }
}

bool Renderer::Initialize(const string& filePath, ID3D11Device* device, ID3D11DeviceContext* dc, ID3D11ShaderResourceView* tex, ConstantBuffer<CB_VS_Transform>& cb_vs_transform)
{
    this->device = device;
    this->dc = dc;

    if (!LoadFile(filePath))
    {
        Debug::Log(L"잘못된 파일 입니다.");
        return false;
    }
    return Initialize(device, dc, tex, cb_vs_transform);
}

void Renderer::SetTexture(ID3D11ShaderResourceView* tex)
{
    texture = tex;
}

void Renderer::Draw(const XMMATRIX vpMat)
{
    UINT offset = 0;
    cb_transform->data.mat = XMMatrixTranspose(gameObject->GetTransform()->GetWorld() * vpMat );
    cb_transform->Update();
    dc->VSSetConstantBuffers(0, 1, cb_transform->GetAddressOf());    
    dc->PSSetShaderResources(0, 1, &texture);
    
    for (Mesh mesh : meshes)
    {
        mesh.Draw();
    }
}

```

</div>

</details>

