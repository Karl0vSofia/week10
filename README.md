<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Handpose Angle Calculation</title>
  
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/handpose"></script>
</head>
<body>
  <video id="video" width="640" height="480" autoplay></video>
  <canvas id="canvas" width="640" height="480"></canvas>

  <script>
   
    const video = document.getElementById('video');
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');

    
    navigator.mediaDevices.getUserMedia({ video: true })
      .then((stream) => {
        video.srcObject = stream;
        video.play();
      });

    
    async function detectHands() {
      const handpose = await handpose.load();
      const predictions = await handpose.estimateHands(video);

    
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

      if (predictions.length > 0) {
        const fingers = predictions[0].landmarks;

       
        function calculateAngle(joint1, joint2, joint3) {
          const v1 = [joint1[0] - joint2[0], joint1[1] - joint2[1]];
          const v2 = [joint3[0] - joint2[0], joint3[1] - joint2[1]];
          const dotProduct = v1[0] * v2[0] + v1[1] * v2[1];
          const magnitude1 = Math.sqrt(v1[0]**2 + v1[1]**2);
          const magnitude2 = Math.sqrt(v2[0]**2 + v2[1]**2);
          const cosTheta = dotProduct / (magnitude1 * magnitude2);
          const angleInRadians = Math.acos(cosTheta);
          const angleInDegrees = (angleInRadians * 180) / Math.PI;
          return angleInDegrees;
        }

      
        const angleThumbIndex = calculateAngle(fingers[4], fingers[5], fingers[6]);
        const angleIndexMiddle = calculateAngle(fingers[8], fingers[9], fingers[10]);
        const angleMiddleRing = calculateAngle(fingers[12], fingers[13], fingers[14]);
        const angleRingPinky = calculateAngle(fingers[16], fingers[17], fingers[18]);

        
        console.log('Angle Thumb-Index:', angleThumbIndex);
        console.log('Angle Index-Middle:', angleIndexMiddle);
        console.log('Angle Middle-Ring:', angleMiddleRing);
        console.log('Angle Ring-Pinky:', angleRingPinky);
      }

      
      requestAnimationFrame(detectHands);
    }

   
    detectHands();
  </script>
</body>
</html>
