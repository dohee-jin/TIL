# 🎣 Throw it or not
### Javascript team project 개인 회고록 - 진도희

## 🚨 트러블 슈팅
### 1. 물고기 생성 인터벌 제어
#### 🚨 문제
메인 인트로 화면에서부터 물고기 버튼이 생성되고 있음
#### 🫧 원인(문제 발생 코드)
물고기 생성 인터벌이 index.html에 접근하자마자 바로 실행되고 있어서 문제 발생.
#### 🚨 추가 문제 발생
```
  intervalId = setInterval(() => {
      if($sea.style.display === 'block'){
          showFish();
      }
  }, 1000);
```
인터벌 내부에 if 조건을 넣어 바다 스테이지 화면이 block 상태 일때만 인터벌이 동작되게 변경<br>
수정되었다고 생각했지만 낚시 모달 종료 후 게임 재시작이 되지 않는 문제 발생<br>
#### 🫧 원인(문제 발생 코드)
```
  intervalId = setInterval(() => {
      if($sea.style.display === 'block'){
          showFish();
      }
  }, 1000);
```
  1. 게임 모달 종료 후 재시작 시 setInetval 중복 실행 되어 재시작에 문제가 발생
  2. if 문에 있는 조건인 UI 상태 변화에 의존하였을 때 원래 예상하던 동작을 기대하기 어려움
#### 🌟 해결
```
// js/module/fish/index.js
function startFishGame() {
      if (intervalId !== null) return; // 중복 방지
      intervalId = setInterval(() => {
          $seaBg.style.animationPlayState = 'running';
          showFish();
      }, 1000);
  }

// js/module/menu/index.js
$gameLoadBtn.addEventListener('click', () => {
    showSeaStage();
    loadGameFromStorage();
    startFishGame();
  });
```
게임 시작 버튼 이벤트 리스너에 startFishGame()을 추가하여 게임이 실행되지 않으면 interval이 실행되지 않게 만들었고,<br>
메인 화면에서 실행되던 showFish() 함수의 실행을 막았다. 또한, intervalId의 상태를 감지하여 interval 중복 실행을<br> 
방지하면서 재시작 문제를 해결했다. 인터벌 재시작 관련 이슈는 관주님께서 도와주셔서 해결할 수 있었다.

### 2. 고정된 위치에 생성되는 물고기
#### 🚨 문제
바다 스테이지 진입 후 랜덤한 위치에 생성되어야 할 물고기가 고정 위치에 생성
#### 🫧 원인(문제 발생 코드)
```
// js
const maxX = $seaWrap.offsetWidth - $fish.offsetWidth - 100;
const maxY = $seaWrap.offsetHeight - $fish.offsetHeight - 100;
```
메인 화면에서 바다 스테이지로 이동할 때 addEventListner( e => {}) 이벤트 리스터 안에서   
display =’none’ → display = “block” 으로 변경 후 화면이 전환되면서    
DOM에 반영하기 전에 offsetWidth, offsetHeight 값을 읽어와 문제 발생  
#### 🌟 해결
화면 변화가 일어나지 않고 전체 뷰포트를 감싸고 있는
```<div id=”throwit-wrap”></div>```의 DOM을 가져온 후   
offsetWidth, offsetHeight 값을 읽어와서 문제를 해결, 하지만 트러블 슈팅을 작성하면서 리플로우 방식을 알게되었는데    
현재 코드는 UI에 의존하는 코드인 반면 리플로우 방식은 UI에 의존하지 않고 문제를 해결할 수 있다는 것을 알게 되었음.   

## 💡 배운점

## ‼️ 개선점
## 💭 소감


