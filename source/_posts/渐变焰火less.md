```less
   @ratio: 1.5;

    .flame {
      position: absolute;
      z-index: 99;
      width: 16px*@ratio;
      height: 26px*@ratio;
      bottom: 410px;
      left: calc(50% - 8px * @ratio);
      background: linear-gradient(180deg, #ee2727 0%, #fba803 100%);
      border-top-left-radius: 8px*@ratio 20px*@ratio;
      border-top-right-radius: 8px*@ratio 20px*@ratio;
      border-bottom-right-radius: 8px*@ratio;
      border-bottom-left-radius: 8px*@ratio;
      animation: wind 10s ease-in-out infinite,
      size 12s ease-in-out infinite,
      flickr 3.5s ease-in-out infinite;
      transform-origin: 50% 100%;
      transform: translate3d(0,0,0);
    }

    @keyframes wind {
      0%, 22%, 49%, 62%, 81%, 100% {
        border-top-left-radius: 2px*@ratio 20px*@ratio;
        border-top-right-radius: 14px*@ratio 20px*@ratio;
        border-bottom-right-radius: 8px*@ratio;
        border-bottom-left-radius: 8px*@ratio;

      }
      14%, 32%, 56%, 70%, 89% {
        border-top-left-radius: 14px*@ratio 20px*@ratio;
        border-top-right-radius: 2px*@ratio 20px*@ratio;
        border-bottom-right-radius: 8px*@ratio;
        border-bottom-left-radius: 8px*@ratio;
      }
    }

    @keyframes size {
      0%, 22%, 49%, 62%, 81%, 100% {
        transform: scale(1, 1);
      }
      14%, 32%, 56%, 70%, 89% {
        transform: scale(0.9, 1.2);
      }
    }

    @keyframes flickr {
      0%, 22%, 49%, 62%, 81%, 100% {
        box-shadow: 0 0 20px 0 rgba(255, 202, 0, 0.7);
      }
      14%, 32%, 56%, 70%, 89% {
        box-shadow: 0 0 20px 0 rgba(255, 202, 0, .8);
      }
    }
```
