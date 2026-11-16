 {
          const proximity = (pushRadius - dist) / pushRadius;
          const pushForce = Math.pow(proximity, 1.25) * 0.016;
          const dirX = dx / dist;
          this.angleAcc += dirX * pushForce;
        }

        this.angleVel += this.angleAcc;
        this.angleVel *= this.damping;
        this.angle += this.angleVel;

        const maxAngle = (80 * Math.PI) / 180;
        if (this.angle > maxAngle) {
          this.angle = maxAngle;
          this.angleVel = Math.min(0, this.angleVel * 0.2);
        } else if (this.angle < -maxAngle) {
          this.angle = -maxAngle;
          this.angleVel = Math.max(0, this.angleVel * 0.2);
        }
      }

      draw(ctx) {
        const knotX = this.originX + Math.sin(this.angle) * this.length;
        const knotY = this.originY + Math.cos(this.angle) * this.length;

        // 밧줄
        ctx.save();
        ctx.beginPath();
        ctx.moveTo(this.originX, this.originY);
        const midX = (this.originX + knotX) / 2 + Math.sin(this.angle) * 6;
        const midY = (this.originY + knotY) / 2;
        ctx.quadraticCurveTo(midX, midY, knotX, knotY);
        ctx.strokeStyle = '#c89d6e';
        ctx.lineWidth = 4;
        ctx.stroke();

        // 매듭
        ctx.beginPath();
        ctx.arc(knotX, knotY, 6, 0, Math.PI * 2);
        ctx.fillStyle = '#9b7145';
        ctx.fill();
        ctx.restore();

        // 캐릭터 렌더링 (밀리는 방향으로 바닥면 들림)
        const bodyAngle = -this.angle * 1.15;

        ctx.save();
        ctx.translate(knotX, knotY);
        ctx.rotate(bodyAngle);

        if (this.img.complete && this.img.naturalWidth > 0) {
          const aspect = this.img.naturalWidth / this.img.naturalHeight;
          const drawH = this.size;
          const drawW = drawH * aspect;
          ctx.drawImage(this.img, -drawW / 2, -drawH * 0.08, drawW, drawH);
        }

        ctx.restore();
      }
    }

    // ----------------------------------------------------
    // 2. 화면 가에(모서리 테두리)를 순찰하는 3번 치이카와 클래스
    // ----------------------------------------------------
    class PerimeterChiikawa {
      constructor(img) {
        this.img = img;
        this.size = Math.min(width, height) * 0.125;

        // 아까(1.2)보단 느리고, 방금(0.65)보단 빠른 최적 속도
        this.speed = 0.92;

        // 엣지 둘레를 따라 도는 거리 매개변수 (distAlongPath)
        this.pathDist = 0;
        this.direction = 1; // 1: 시계 방향, -1: 반시계 방향

        this.state = 'walk'; // walk, idle, jump
        this.stateTimer = 180;
        this.walkStep = 0;

        // 점프 변수
        this.jumpOffset = 0;
        this.jumpVy = 0;

        this.x = 0;
        this.y = 0;
        this.baseAngle = 0; // 벽에 맞춘 회전 각도
      }

      onResize() {
        this.size = Math.min(width, height) * 0.125;
      }

      // 모니터 가장자리 테두리 둘레 계산
      getPerimeterBounds() {
        const pad = this.size * 0.52;
        const left = pad;
        const right = width - pad;
        const top = Math.max(pad, 45); // 상단 나무 바 바로 아래
        const bottom = height - pad;

        const w = right - left;
        const h = bottom - top;
        const total = (w + h) * 2;

        return { left, right, top, bottom, w, h, total };
      }

      // 둘레 거리(dist)를 화면 좌표와 벽면 회전각으로 변환
      getPositionFromDist(dist) {
        const { left, right, top, bottom, w, h, total } = this.getPerimeterBounds();
        let d = ((dist % total) + total) % total;

        // 1. 하단 바닥면 (left -> right)
        if (d < w) {
          return { x: left + d, y: bottom, angle: 0 };
        }
        d -= w;

        // 2. 우측 벽면 (bottom -> top)
        if (d < h) {
          return { x: right, y: bottom - d, angle: -Math.PI / 2 };
        }
        d -= h;

        // 3. 상단 천장면 (right -> left)
        if (d < w) {
          return { x: right - d, y: top, angle: Math.PI };
        }
        d -= w;

        // 4. 좌측 벽면 (top -> bottom)
        return { x: left, y: top + d, angle: Math.PI / 2 };
      }

      // 클릭 여부 판정
      isClicked(mx, my) {
        const aspect = (this.img.naturalWidth && this.img.naturalHeight) ? (this.img.naturalWidth / this.img.naturalHeight) : 1;
        const radius = (this.size * Math.max(1, aspect)) * 0.55;
        return Math.hypot(mx - this.x, my - this.y) <= radius;
      }

      // 클릭 시 점프
      triggerJump() {
        this.state = 'jump';
        this.jumpVy = -7.8;
      }

      update() {
        this.stateTimer--;

        // 가끔 멈춰서 두리번거리거나 방향 반전
        if (this.stateTimer <= 0 && this.state !== 'jump') {
          const rand = Math.random();
          if (rand < 0.7) {
            this.state = 'walk';
            this.stateTimer = 180 + Math.floor(Math.random() * 220); // 3초 ~ 7초 동안 걷기
            if (Math.random() < 0.25) {
              this.direction *= -1; // 가끔 반대 방향으로 턴
            }
          } else {
            this.state = 'idle';
            this.stateTimer = 70 + Math.floor(Math.random() * 90); // 1.2초 ~ 2.5초 휴식
          }
        }

        // 점프 물리
        if (this.jumpVy !== 0 || this.jumpOffset !== 0) {
          this.jumpOffset += this.jumpVy;
          this.jumpVy += 0.42;

          if (this.jumpOffset >= 0) {
            this.jumpOffset = 0;
            this.jumpVy = 0;
            if (this.state === 'jump') this.state = 'walk';
          }
        }

        // 둘레 이동
        if (this.state === 'walk') {
          this.pathDist += this.speed * this.direction;
          this.walkStep += 0.12; // 경쾌하게 아장거리는 걸음걸이
        }

        // 현재 테두리 위치 계산
        const pos = this.getPositionFromDist(this.pathDist);
        this.x = pos.x;
        this.y = pos.y;
        this.baseAngle = pos.angle;
      }

      draw(ctx) {
        if (!this.img.complete || this.img.naturalWidth <= 0) return;

        ctx.save();

        const aspect = this.img.naturalWidth / this.img.naturalHeight;
        const drawH = this.size;
        const drawW = drawH * aspect;

        let tilt = 0;
        let scaleY = 1;
        let scaleX = 1;

        if (this.state === 'walk') {
          tilt = Math.sin(this.walkStep) * 0.11;
          scaleY = 1 + Math.sin(this.walkStep * 2) * 0.045;
          scaleX = 1 - Math.sin(this.walkStep * 2) * 0.035;
        } else if (this.jumpOffset < 0) {
          scaleY = 1.15;
          scaleX = 0.9;
        } else if (this.state === 'idle') {
          scaleY = 1 + Math.sin(Date.now() * 0.005) * 0.035;
        }

        ctx.translate(this.x, this.y);
        ctx.rotate(this.baseAngle);

        // 점프 시 벽면에서 수직으로 튀어오르는 방향 적용
        ctx.translate(0, this.jumpOffset);

        // 걷는 진행 방향에 따른 좌우 반전
        const facing = this.direction >= 0 ? 1 : -1;
        ctx.scale(facing * scaleX, scaleY);
        ctx.rotate(tilt);

        ctx.drawImage(this.img, -drawW / 2, -drawH / 2, drawW, drawH);

        ctx.restore();
      }
    }

    let chiikawas = [];
    let crawler = null;

    function initPendulums() {
      chiikawas = [];
      const count = images.length;
      const spacing = width / (count + 1);

      for (let i = 1; i <= count; i++) {
        const x = spacing * i;
        const length = height * 0.28 + (i % 2 === 0 ? 38 : -25);
        const size = Math.min(width, height) * 0.18;
        chiikawas.push(new HangingChiikawa(x, length, size, images[i - 1]));
      }

      if (!crawler) {
        crawler = new PerimeterChiikawa(images[2]); // 3번 치이카와
      }
    }

    initPendulums();

    // 클릭 시 테두리를 걷는 치이카와 점프
    window.addEventListener('mousedown', (e) => {
      if (crawler && crawler.isClicked(e.clientX, e.clientY)) {
        crawler.triggerJump();
      }
    });

    function animate() {
      ctx.clearRect(0, 0, width, height);

      // 상단 지지대 봉
      ctx.beginPath();
      ctx.moveTo(0, 5);
      ctx.lineTo(width, 5);
      ctx.strokeStyle = '#cbb3a3';
      ctx.lineWidth = 10;
      ctx.stroke();

      // 상단 매달린 4마리
      chiikawas.forEach((c) => {
        c.update();
        c.draw(ctx);
      });

      // 테두리 둘레를 아장아장 걷는 3번 치이카와
      if (crawler) {
        crawler.update();
        crawler.draw(ctx);
      }

      requestAnimationFrame(animate);
    }

    animate();
  </script>
</body>
</html>
