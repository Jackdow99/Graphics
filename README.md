# 병 안에 배 무드등

```
import React, {
  useRef,
  useState,
  useEffect,
  Suspense,
  useMemo,
} from "react";

import { Canvas, useFrame } from "@react-three/fiber";
import { OrbitControls, useGLTF } from "@react-three/drei";
import * as THREE from "three";

function DownloadedShip() {
  const shipRef = useRef();

  let scene;

  try {
    const gltf = useGLTF("/Ship.glb");
    scene = gltf.scene;
  } catch (error) {
    console.error("배 모델 로드 실패:", error);
    scene = null;
  }

  useFrame((state) => {
    const time = state.clock.getElapsedTime();

    if (shipRef.current) {
      shipRef.current.rotation.z = Math.sin(time * 1.8) * 0.08;
      shipRef.current.rotation.x = Math.cos(time * 1.2) * 0.04;

      shipRef.current.position.y =
        -0.12 +
        Math.sin(time * 2.0) * 0.02 +
        Math.cos(time * 1.4) * 0.01;
    }
  });

  if (scene) {
    return (
      <primitive
        ref={shipRef}
        object={scene}
        scale={0.05}
        position={[0.1, -0.12, 0]}
      />
    );
  }

  return (
    <mesh ref={shipRef} position={[0, -0.12, 0]}>
      <boxGeometry args={[0.3, 0.1, 0.15]} />
      <meshStandardMaterial color="#8B4513" />
    </mesh>
  );
}

function CodeWater() {
  const surfaceRef = useRef();

  const surfaceGeometry = useMemo(() => {
    return new THREE.CircleGeometry(0.42, 64);
  }, []);

  useFrame((state) => {
    const time = state.clock.getElapsedTime();

    if (!surfaceRef.current) return;

    const pos = surfaceRef.current.geometry.attributes.position;

    for (let i = 0; i < pos.count; i++) {
      const x = pos.getX(i);
      const y = pos.getY(i);

      const wave =
        Math.sin(x * 8 + time * 2) * 0.02 +
        Math.cos(y * 8 + time * 1.5) * 0.02;

      pos.setZ(i, wave);
    }

    pos.needsUpdate = true;
    surfaceRef.current.geometry.computeVertexNormals();
  });

  return (
    <group>
      {/* 물 본체 */}
      <mesh position={[0, -0.3, 0]}>
        <cylinderGeometry args={[0.42, 0.42, 0.25, 64]} />
        <meshPhysicalMaterial
          color="#0088dd"
          transparent
          opacity={0.55}
          transmission={0.5}
          roughness={0.1}
        />
      </mesh>

      {/* 물 표면 */}
      <mesh
        ref={surfaceRef}
        geometry={surfaceGeometry}
        position={[0, -0.18, 0]}
        rotation={[-Math.PI / 2, 0, 0]}
      >
        <meshPhysicalMaterial
          color="#00aaff"
          transparent
          opacity={0.8}
          transmission={0.8}
          roughness={0.05}
        />
      </mesh>
    </group>
  );
}

function MainMoodLight() {
  const [isOn, setIsOn] = useState(true);
  const [lightColor, setLightColor] = useState("#ffddaa");

  useEffect(() => {
    const handleKeyDown = (e) => {
      if (e.key === "1") setLightColor("#ffddaa");
      if (e.key === "2") setLightColor("#ff6600");
      if (e.key === "3") setLightColor("#2299ff");
    };

    window.addEventListener("keydown", handleKeyDown);

    return () => {
      window.removeEventListener("keydown", handleKeyDown);
    };
  }, []);

  return (
    <group position={[0, -0.2, 0]}>
      {/* 받침대 */}
      <mesh position={[0, -0.6, 0]}>
        <cylinderGeometry args={[0.55, 0.6, 0.15, 32]} />
        <meshStandardMaterial color="#3a2214" roughness={0.8} />
      </mesh>

      {/* 조명 */}
      {isOn && (
        <>
          <pointLight
            position={[0, 0.1, 0]}
            intensity={5}
            distance={5}
            decay={1.5}
            color={lightColor}
          />

          <pointLight
            position={[0, 0.1, 0]}
            intensity={2}
            distance={2}
            color={lightColor}
          />
        </>
      )}

      {/* 병 */}
      <group
        onClick={(e) => {
          e.stopPropagation();
          setIsOn(!isOn);
        }}
      >
        {/* 유리 몸통 */}
        <mesh position={[0, 0, 0]}>
          <cylinderGeometry args={[0.45, 0.45, 0.8, 64, 1, true]} />
          <meshPhysicalMaterial
            color="#ffffff"
            transparent
            opacity={0.12}
            transmission={1}
            roughness={0}
            thickness={0.3}
            side={THREE.DoubleSide}
          />
        </mesh>

        {/* 목 부분 */}
        <mesh position={[0, 0.45, 0]}>
          <cylinderGeometry args={[0.2, 0.45, 0.1, 32, 1, true]} />
          <meshPhysicalMaterial
            color="#ffffff"
            transparent
            opacity={0.12}
            transmission={1}
            roughness={0}
            side={THREE.DoubleSide}
          />
        </mesh>

        <mesh position={[0, 0.53, 0]}>
          <cylinderGeometry args={[0.2, 0.2, 0.06, 32, 1, true]} />
          <meshPhysicalMaterial
            color="#ffffff"
            transparent
            opacity={0.12}
            transmission={1}
            roughness={0}
            side={THREE.DoubleSide}
          />
        </mesh>

        {/* 코르크 마개 */}
        <mesh position={[0, 0.58, 0]}>
          <cylinderGeometry args={[0.19, 0.18, 0.05, 16]} />
          <meshStandardMaterial color="#a07855" />
        </mesh>

        {/* 배 */}
        <Suspense
          fallback={
            <mesh position={[0, -0.12, 0]}>
              <boxGeometry args={[0.2, 0.1, 0.1]} />
              <meshBasicMaterial color="orange" wireframe />
            </mesh>
          }
        >
          <DownloadedShip />
        </Suspense>

        {/* 물 */}
        <CodeWater />
      </group>
    </group>
  );
}

export default function App() {
  return (
    <div
      style={{
        width: "100vw",
        height: "100vh",
        background: "#05060c",
        position: "relative",
      }}
    >
      <div
        style={{
          position: "absolute",
          top: 20,
          left: 20,
          color: "white",
          zIndex: 10,
          fontFamily: "sans-serif",
          pointerEvents: "none",
        }}
      >
        <p>드래그 : 카메라 회전</p>
        <p>유리병 클릭 : 전원 ON/OFF</p>
        <p>1 = 노랑 / 2 = 주황 / 3 = 파랑</p>
      </div>

      <Canvas camera={{ position: [0, 0.8, 2], fov: 45 }}>
        <ambientLight intensity={0.2} />

        <directionalLight
          position={[2, 4, 3]}
          intensity={0.5}
        />

        <MainMoodLight />

        <OrbitControls
          enableDamping
          dampingFactor={0.05}
          minPolarAngle={0.9}
          maxPolarAngle={1.4}
        />
      </Canvas>
    </div>
  );
}
```
