<template>
  <div></div>
</template>

<script lang = "ts">
import Vue from 'vue';
import Component from 'vue-class-component';
import {
    PerspectiveCamera, OrthographicCamera, Vector3,
    CameraHelper, Quaternion, Vector2, Spherical, Camera} from 'three';

export class CameraControls extends Vue {

    // Screen Dimensions
    private screen: { left: number; top: number; width: number; height: number };

    // Camera Properties
    private mainPerspectiveCamera: PerspectiveCamera;

    private objPerspectiveCamera: PerspectiveCamera;
    private objPerspectiveCameraHelper: CameraHelper;

    private objOrthographicCamera: OrthographicCamera;
    private objOrthographicCameraHelper: CameraHelper;

    private isObjCameraPerspective: boolean;

    // Perspective Camera Properties
    private nearPlane: number;
    private farPlane: number;
    private fov: number;

    // Orthographic Camera Properties
    private frustumSize: number;

    // Object Camera Properties
    private nearPlaneObj: number;
    private farPlaneObj: number;

    private posXObj: number;
    private posYObj: number;
    private posZObj: number;

    private screenAspectRatio: number;

    // Orbit Controls Params
    private minPolarAngle: number;
    private maxPolarAngle: number;

    private minAzimuthAngle: number;
    private maxAzimuthAngle: number;

    private rotateSpeed: number;

    private position0: Vector3;

    private target: Vector3;
    private target0: Vector3;

    private spherical: Spherical;
    private sphericalDelta: Spherical;

    private scale: number;

    private rotateStart: Vector2;
    private rotateEnd: Vector2;
    private rotateDelta: Vector2;

    private minDistance: number;
    private maxDistance: number;

    constructor() {

        super();

        this.screen = {
            left: 0,
            top: 0,
            width: 600,
            height: 300,
        };

        this.nearPlane = 100;
        this.farPlane = 10000;
        this.fov = 40;
        this.frustumSize = 50;

        // this.screenAspectRatio = 2 * this.screen.width / this.screen.height;
        this.screenAspectRatio = this.screen.width / this.screen.height;

        this.mainPerspectiveCamera = new PerspectiveCamera(this.fov, this.screenAspectRatio,
            this.nearPlane, this.farPlane);

        this.mainPerspectiveCamera.position.z = 700;

        this.nearPlaneObj = 30;
        this.farPlaneObj = 400;

        this.posXObj = 0;
        this.posYObj = 10;
        this.posZObj = 400;

        this.objPerspectiveCamera = new PerspectiveCamera(this.fov, this.screenAspectRatio,
            this.nearPlaneObj, this.farPlaneObj);

        // look at and dir should be opposite to each other
        this.objPerspectiveCamera.lookAt(-this.posXObj, -this.posYObj, -this.posZObj);
        this.objPerspectiveCamera.position.set(this.posXObj, this.posYObj, this.posZObj);

        this.objPerspectiveCameraHelper = new CameraHelper(this.objPerspectiveCamera);

        this.objOrthographicCamera = new OrthographicCamera(- this.frustumSize, this.frustumSize,
            this.frustumSize, - this.frustumSize, this.nearPlaneObj, this.farPlaneObj);

        this.objOrthographicCamera.lookAt(-this.posXObj, -this.posYObj, -this.posZObj);
        this.objOrthographicCamera.position.set(this.posXObj, this.posYObj, this.posZObj);

        this.objOrthographicCameraHelper = new CameraHelper(this.objOrthographicCamera);

        this.isObjCameraPerspective = true;

        this.target = new Vector3();
        this.target0 = this.target.clone();

        // Orbit Controls
        this.minPolarAngle = 0;
        this.maxPolarAngle = Math.PI * 0.5;

        this.minAzimuthAngle = -Infinity;
        this.maxAzimuthAngle = Infinity;

        this.rotateSpeed = 1.0;

        this.position0 = this.mainPerspectiveCamera.position.clone();

        this.spherical = new Spherical();
        this.sphericalDelta = new Spherical();

        this.scale = 1;
        this.rotateStart = new Vector2();
        this.rotateEnd = new Vector2();
        this.rotateDelta = new Vector2();

        this.minDistance = 100;
        this.maxDistance = 5000;
    }

    protected setIsObjCameraPerspective(value: boolean) {

        this.isObjCameraPerspective = value;
    }

    protected getMainCamera() {

        return this.mainPerspectiveCamera;
    }

    protected getObjectPerspectiveCameraHelper() {

        return this.objPerspectiveCameraHelper;
    }

    protected getObjectOrthographicCameraHelper() {

        return this.objOrthographicCameraHelper;
    }

    protected getObjectPerspectiveCamera() {

        return this.objPerspectiveCamera;
    }

    protected getObjectOrthographicCamera() {

        return this.objOrthographicCamera;
    }

    // Slider operation on Object Camera
    protected translateCameraX(valX: number) {

        this.objPerspectiveCamera.position.x = - valX * 50;
        this.objOrthographicCamera.position.x = - valX * 50;
    }

    protected translateCameraY(valY: number) {

        this.objPerspectiveCamera.position.y = - valY * 50;
        this.objOrthographicCamera.position.y = - valY * 50;
    }

    protected translateCameraZ(valZ: number) {

        this.objPerspectiveCamera.position.z = - valZ * 50;
        this.objOrthographicCamera.position.z = - valZ * 50;
    }

    protected changeCameraFOV(valY: number) {

        if (this.isObjCameraPerspective) {

            this.fov = valY * 10;
            this.objPerspectiveCamera.fov = this.fov;
        } else {

            this.frustumSize = valY * 10;
            this.objOrthographicCamera.left = - this.frustumSize;
            this.objOrthographicCamera.right = this.frustumSize;
            this.objOrthographicCamera.top = this.frustumSize;
            this.objOrthographicCamera.bottom = - this.frustumSize;
        }
    }

    protected changeCameraFar(valZ: number) {

        this.farPlaneObj = valZ * 100;
        this.objPerspectiveCamera.far = this.farPlaneObj;
        this.objOrthographicCamera.far = this.farPlaneObj;
    }

    // Main Camera Operations on Mouse Events
    protected startRotationOnMouseDown(event: MouseEvent) {
        this.rotateStart.set(event.clientX, event.clientY);
    }

    protected rotateOnMouseMove(event: MouseEvent, element: HTMLCanvasElement) {

        this.rotateEnd.set(event.clientX, event.clientY);
        this.rotateDelta.subVectors(this.rotateEnd, this.rotateStart).multiplyScalar(this.rotateSpeed);

        this.rotateLeft(2 * Math.PI * this.rotateDelta.x / element.clientHeight);
        this.rotateUp(2 * Math.PI * this.rotateDelta.y / element.clientHeight);

        this.rotateStart.copy(this.rotateEnd);

        this.updateAndRotateMainCamera();
    }

    protected resetMainCamera() {

        this.target.copy(this.target0);

        this.mainPerspectiveCamera.position.copy(this.position0);
        this.mainPerspectiveCamera.updateProjectionMatrix();

        this.updateAndRotateMainCamera();
    }

    // ported from Orbitcontrols in three.js
    protected updateAndRotateMainCamera() {

        const position = this.mainPerspectiveCamera.position;

        const offset = new Vector3();
        offset.copy(position).sub(this.target);

        const quaternion = new Quaternion().setFromUnitVectors(this.mainPerspectiveCamera.up, new Vector3(0, 1, 0));
        const quaternionInverse = quaternion.clone().inverse();

        offset.applyQuaternion(quaternion);

        this.spherical.setFromVector3(offset);

        this.spherical.theta += this.sphericalDelta.theta;
        this.spherical.phi += this.sphericalDelta.phi;

        this.spherical.theta = Math.max(this.minAzimuthAngle, Math.min(this.maxAzimuthAngle, this.spherical.theta));
        this.spherical.phi = Math.max(this.minPolarAngle, Math.min(this.maxPolarAngle, this.spherical.phi));

        this.spherical.makeSafe();

        this.spherical.radius *= this.scale;

        this.spherical.radius = Math.max(this.minDistance, Math.min(this.maxDistance, this.spherical.radius));

        offset.setFromSpherical(this.spherical);
        offset.applyQuaternion(quaternionInverse);

        position.copy(this.target).add(offset);

        this.mainPerspectiveCamera.lookAt(this.target);

        this.sphericalDelta.set(0, 0, 0);

        this.scale = 1;
    }

    private rotateLeft(angle: number) {

        this.sphericalDelta.theta -= angle;
    }

    private rotateUp(angle: number) {

        this.sphericalDelta.phi -= angle;
    }
}

export default CameraControls;
</script>