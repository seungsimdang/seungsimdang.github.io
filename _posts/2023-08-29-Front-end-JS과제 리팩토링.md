---
published: true
title: '2023-08-29-Front-End-JS과제 리팩토링'
categories:
  - Front-End
tags:
  - Front-End
toc: true
toc_sticky: true
toc_label: 'Front-End'
---

# 이전 코드 VS 리펙토링 코드

### 이전 코드

```css
html {
  font-size: 14px;
}
```

### 리팩토링 코드

```css
html {
  font-size: 62.5%;
}
```

`font-size`를 62.5%로 설정하면 웹 접근성에도 도움이 되고 `10px` 값으로 계산이 되어서 `rem` 계산할 때 좀 더 편할것이라는 태관님의 리뷰를 참고하여 위와 같이 리팩토링하였다.

<br>

### 이전 코드

```javascript
// 페이지 로드 시 직원 정보 표시
window.onload = async () => {
  // 직원 정보 조회
  const docRef = doc(db, 'users', userID);
  await getDoc(docRef).then((docSnap) => {
    // 스켈레톤 레이아웃 삭제
    document.querySelectorAll('.skeleton').forEach((skeleton) => {
      skeleton.classList.remove('skeleton');
    });
    // 값이 존재하면 직원 정보 표시
    if (docSnap.exists()) {
      profileImg.src = docSnap.data().profile;
      alert('존재하지 않는 직원입니다.');
      location.href = '/';
    }
  });
};
```

### 리팩토링 코드

```javascript
async function fetchDataAndDisplay(docRef) {
  try {
    const docSnap = await getDoc(docRef);

    // 스켈레톤 레이아웃 삭제
    document.querySelectorAll('.skeleton').forEach((skeleton) => {
      skeleton.classList.remove('skeleton');
    });

    // 값이 존재하면 직원 정보 표시
    if (docSnap.exists()) {
      profileImg.src = docSnap.data().profile;
      imgTextInput.value = docSnap.data().profile;
      nameInput.value = docSnap.data().name;
      emailInput.value = docSnap.data().email;
      phoneInput.value = docSnap.data().phone;
    } else {
      alert('존재하지 않는 직원입니다.');
      location.href = '/';
    }
  } catch (error) {
    console.error('Error fetching data:', error);
  }
}

// 페이지 로드 시 직원 정보 표시
window.onload = async () => {
  const docRef = doc(db, 'users', userID);
  await fetchDataAndDisplay(docRef);
};
```

기존 코드에서는 `window`가 `load`될 때 `async/await`를 사용하여 document를 가져온 후 then 메소드 체이닝을 이용하였다면, 리팩토링 코드에서는 별도의 `async`함수를 구성하여 `await`의 형태로 호출하는 방식으로 변경하였다. 또한, `try/catch`를 사용하여 에러를 핸들링하여 안정성을 높이는 방향으로 리팩토링하였다.

<br>

### 이전 코드

```javascript
// 버튼을 클릭 시 삭제
const deleteBtn = document.querySelector('.delete-btn');
deleteBtn.addEventListener('click', async (e) => {
  if (deleteList.length > 0) {
    if (window.confirm('삭제하시겠습니까?')) {
      deleteList.forEach(async (id) => {
        await deleteDoc(doc(db, 'users', id)).then(() => {
          alert('삭제되었습니다.');
          location.reload();
        });
      });
    } else {
      alert('취소되었습니다.');
    }
  }
});
```

### 리팩토링 코드

```javascript
async function deleteDocuments() {
  try {
    const deletePromises = deleteList.map((id) =>
      deleteDoc(doc(db, 'users', id))
    );
    await Promise.all(deletePromises);
    alert('삭제되었습니다.');
    location.reload();
  } catch (error) {
    console.error('Error deleting documents:', error);
  }
}

// 버튼을 클릭 시 삭제
const deleteBtn = document.querySelector('.delete-btn');
deleteBtn.addEventListener('click', async (e) => {
  if (deleteList.length > 0) {
    if (window.confirm('삭제하시겠습니까?')) {
      await deleteDocuments();
    } else {
      alert('취소되었습니다.');
    }
  } else {
    alert('삭제할 직원을 선택하세요.');
  }
});
```

기존 코드에서 콜백함수가 길어져 가독성이 떨어지는 이슈가 있었는데, 별도의 `deleteDocuments`함수로 분리하여 가독성을 높였다. 또한, 기존 `forEach`문 내에서 `async/await`를 이용하였는데, `Promise.all`을 이용하여 `deleteList` 내의 해당하는 id의 document를 삭제하도록 리팩토링하였다.

<br>

### 이전 코드

```javascript
export const changeProfile = (userID) => {
  const imageInputEl = document.getElementById('profilePic');

  imageInputEl.addEventListener('change', () => {
    if (imgTextInput.value) {
      deleteData(imgTextInput.value);
    }
    const file = imageInputEl.files[0];
    const storageRef = ref(storage, 'profile/' + Math.random() + file.name);
    // storage에 사진 저장
    uploadBytes(storageRef, file).then(() => {
      // storage에 저장된 사진 url 가져오기
      getDownloadURL(storageRef).then(async (url) => {
        // 프로필 이미지 url input에 저장
        imgTextInput.value = url;

        // 프로필 이미지 변경
        document.getElementById('profileImg').src = url;

        // 프로필 이미지 삭제 버튼 표시
        imgRemoveBtn.classList.remove('hidden');

        // 정보 수정일 경우에는 firestore에서 바로 수정
        if (userID) {
          const batch = writeBatch(db);
          const IdRef = doc(db, 'users', userID);
          batch.update(IdRef, { profile: imgTextInput.value });
          await batch.commit();
        }
      });
    });
  });
};
```

### 리팩토링 코드

```javascript
// input 파일이 바뀌면 firebase Storage에 저장하고 화면에 표시
export const changeProfile = (userID) => {
  const imageInputEl = document.getElementById('profilePic');

  imageInputEl.addEventListener('change', async () => {
    const file = imageInputEl.files[0];

    if (file) {
      const storageRef = ref(storage, 'profile/' + Math.random() + file.name);

      try {
        const [uploadResult, url] = await Promise.all([
          // storage에 사진 저장
          uploadBytes(storageRef, file),
          // storage에 저장된 사진 url 가져오기
          getDownloadURL(storageRef),
        ]);

        // 프로필 이미지 url input에 저장
        imgTextInput.value = url;
        // 프로필 이미지 변경
        document.getElementById('profileImg').src = url;
        // 프로필 이미지 삭제 버튼 표시
        imgRemoveBtn.classList.remove('hidden');

        // 정보 수정일 경우에는 firestore에서 바로 수정
        if (userID) {
          const batch = writeBatch(db);
          const idRef = doc(db, 'users', userID);
          batch.update(idRef, { profile: url });
          await batch.commit();
        }
      } catch (error) {
        console.error('Error uploading image:', error);
      }
    }
  });
};
```

코드 리뷰 중 기존 코드의 `uploadBytes` 함수와 `getDownloadURL` 함수를 비동기 처리할 때 선후 관계 및 서로 결과 값에 영향을 받지 않는 것 같다는 지적이 있었다. 이후, 병렬처리를 통해 코드도 줄이고 성능도 최적화 할 수 있을 것 같다는 피드백을 해주셔서 `Promise.all([uploadBytes(params), getDownloadURL(params)])`의 형태로 리팩토링하였다.

<br>

# 추후 개선할 점

민석님이 지적해주신 등록전 미리보기 이미지가 안뜨는 이슈에 대한 해결 방안으로 `Promise.all`을 이용하여 병렬처리를 해주었지만,  
![image](https://github.com/seungsimdang/seungsimdang.github.io/blob/master/_images/photo_manager1.png?raw=true)  
위와 같은 에러를 마주했다.

<br>

![image](https://github.com/seungsimdang/seungsimdang.github.io/blob/master/_images/photo_manager2.png?raw=true)  
firebase console storage 상에는 위와 같이 잘 저장 되어있는데 `getDownloadURL` 함수에서 `storageRef` 주소에 접근하였을 때 저장된 데이터를 못 찾는것 같다. 이 이슈에 대해서는 좀 더 검색이나 자문을 구해보고 개선하도록 하겠다.

<br>

# 느낀점

솔직히 JS 과제를 진행하기 전 수강했던 온라인 강의와 실시간 강의의 내용만을 보고 과제를 수행하려니 만만치 않았다. firebase의 경우도 처음 사용해보는거라 이런저런 수행착오도 많았다. 하지만, 열심히 검색하고 찾아보면서 결국 과제를 완성할 수 있어서 좋았고, 리팩토링까지 나름 성공적으로 끝낼 수 있어서 뿌듯하다.
