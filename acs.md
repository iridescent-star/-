domain GateState { CLOSED, OPENED, FAILED } //Gate���״̬��
domain MotorPumpState { STANDBY , WORKING , FAILED } //Motor���״̬��

//in��input ����
//out��output ���

//Gate��
class Gate
	GateState _state (init = CLOSED);
	Boolean in , out (reset = false);
    event open(delay = Dirac(0) , expectation= gamma );
	event close(delay = Dirac(0) , expectation= gamma );
	event failure (delay = exponential( lambda ));
	event repair (delay = exponential (1/ tau ));
	parameter Real lambda = 1.0e-4;
	parameter Real tau = 8;
	parameter Real gamma = 0.02;
	transition
		open: _state == CLOSED -> _state := OPENED;
		close: _state == OPENED -> _state := CLOSED;
		failure : _state == OPENED -> _state := FAILED ;
		repair : _state == FAILED -> _state := CLOSED ;
	assertion
		out := in and _state == OPENED;
end

//Motor��
class Motor
	MotorPumpState _state (init = STANDBY );
	Boolean demand , in , out (reset = false);
	event start (delay = Dirac(0) , expectation= gamma );
	event failureOnDemand (delay = Dirac(0) , expectation=1 - gamma );
	event stop (delay = Dirac (0));
	event failure (delay = exponential( lambda ));
	event repair (delay = exponential (1/ tau ));
	parameter Real lambda = 1.0e-4;
	parameter Real tau = 8;
	parameter Real gamma = 0.02;
	transition
		start : demand and _state == STANDBY -> _state := WORKING ;
		failureOnDemand : demand and _state == STANDBY -> _state := FAILED ;
		stop : not demand and _state == WORKING -> _state := STANDBY ;
		failure : _state == WORKING -> _state := FAILED ;
		repair : _state == FAILED -> _state := STANDBY ;
	assertion
		out := in and _state == WORKING ;
end

//�����޸����
class NonRepairableComponent
	Boolean working (init = true);
	Boolean in, out(reset = false);
    event failure (delay = exponential(lambda));
    parameter Real lambda = 1.0e-5;
    transition
		failure: working -> working := false;
	assertion
		out := in and working;
end
//���޸����
class RepairableComponent
	extends NonRepairableComponent;
	parameter Real pmu = 1.0e-2;
    event repair (delay = exponential(pmu));
    transition
		repair: not working -> working := true;
end

/*����*/
//����ҶƬ
class TurbineLeaves
    NonRepairableComponent leaf1,leaf2,leaf3;
	Boolean in, out (reset = false);
	assertion
		leaf1.in := in;
		leaf2.in := in;
		leaf3.in := in;
		out := leaf1.out and leaf2.out and leaf3.out;
end
//���컷
class TurbineRing
	NonRepairableComponent component1, component2;
	Boolean in, out (reset = false);
	assertion
		component1.in := in;
		component2.in := in;
		out := component1.out and component2.out;
end
//�����Ͽ�
class TurbineVolute
	NonRepairableComponent volute;
	Boolean in, out (reset = false);
	assertion
		volute.in := in;
		out := volute.out;
end
//�������
class Bearing
	NonRepairableComponent component1, component2;
	Boolean in, out (reset = false);
	assertion
		component1.in := in;
		component2.in := in;
		out := component1.out and component2.out;
end
//���������
class TurbineBearings
	Bearing radial, thrust;
	Boolean in, out (reset = false);
    assertion
		radial.in := in;
		thrust.in := in;
		out := radial.out and thrust.out;
end
//�������
class HalfHousing
	NonRepairableComponent component1, component2;
	Boolean in, out (reset = false);
	assertion
		component1.in := in;
		component2.in := in;
		out := component1.out and component2.out;
end
//���ֿ���
class TurbineHousing
	Bearing left, right;
	Boolean in, out (reset = false);
    assertion
		left.in := in;
		right.in := in;
		out := left.out and right.out;
end
//�����ж��ܷ��
class TurbineSealingPlate
	NonRepairableComponent component1, component2;
	Boolean in, out (reset = false);
	assertion
		component1.in := in;
		component2.in := in;
		out := component1.out and component2.out;
end
//�����ܷ�Ȧ
class TurbineSealingRing
	NonRepairableComponent component1, component2;
	Boolean in, out (reset = false);
	assertion
		component1.in := in;
		component2.in := in;
		out := component1.out and component2.out;
end
//���������
class TurbineAxis
	NonRepairableComponent component1, component2;
	Boolean in, out (reset = false);
	assertion
		component1.in := in;
		component2.in := in;
		out := component1.out and component2.out;
end
//����ת�ٲ���װ��
class TurbineSpeadMeasure
	NonRepairableComponent component1, component2;
	Boolean in, out (reset = false);
	assertion
		component1.in := in;
		component2.in := in;
		out := component1.out and component2.out;
end

//Turbine��
class Turbine
    TurbineLeaves TL;
	TurbineRing TR;
	TurbineVolute TV;
	TurbineBearings TB;
	TurbineHousing TH;
	TurbineSealingPlate TSP;
	TurbineSealingRing TSR;
	TurbineAxis TA;
	TurbineSpeadMeasure TSM;
    Boolean in, out (reset = false);
	assertion
		TL.in := in;
		TR.in := in;
		TV.in := in;
		TB.in := in;
		TH.in := in;
		TSP.in := in;
		TSR.in := in;
		TA.in := in;
		TSM.in := in;
		out := TL.out and TR.out and TV.out and TB.out and TH.out and TSP.out and TSR.out and TA.out and TSM.out;
end

block acs
	Gate gate1, gate2, gate3; //�������
	Gate paCtr; //ѹ��������
	Gate ctr1, ctr2; //���ڻ���
	Motor air2air; //����-����������
	Motor gas2air; //ȼ��-����������
	RepairableComponent dropwater1, dropwater2; //��ˮ��
    NonRepairableComponent pascaSignal; //ѹ���ź���
	Turbine T; //����
	NonRepairableComponent temperatureSignal; //�¶ȴ�����
	RepairableComponent flow1, flow2; //����������
	Boolean input, outputElec, outputSeat (reset = false); //���루�����������������Ӳ�������������
	assertion
		gate1.in := input;
		paCtr.in := gate1.out;
		air2air.in := paCtr.out; ctr1.in := paCtr.out;
		gas2air.in := air2air.out; ctr2.in := air2air.out;
		dropwater1.in := gas2air.out;
		pascaSignal.in := dropwater1.out;
		T.in := pascaSignal.out;
		temperatureSignal.in := T.out and ctr2.out;
		dropwater2.in := temperatureSignal.out;
		flow1.in := dropwater2.out or ctr1.out;
		gate2.in := flow1.out;
		gate3.in := dropwater2.out or ctr1.out;
		flow2.in := gate3.out;
		outputElec := gate2.out;
		outputSeat := flow2.out;
end
