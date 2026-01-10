FUNCTION_BLOCK FB_UniversalVFD_CiA402
VAR_INPUT
    xEnable         : BOOL;  // Reglerfreigabe (Power)
    xRun            : BOOL;  // Start/Stop (Zustand 0x7F)
    xReset          : BOOL;  // Fehler quittieren (Bit 7: Fault Reset)
    xHalt           : BOOL;  // Achse anhalten (Bit 8: Halt)
    iTargetSpeed    : INT;   // Sollgeschwindigkeit (6042h)
    uiStatusWord    : UINT;  // Rücklesewert vom Antrieb (6041h)
END_VAR

VAR_OUTPUT
    uiControlWord   : UINT;  // Befehl an Antrieb (6040h)
    iModeOfOp       : BYTE;  // Modus (VFD / Velocity = 2) (6060h)
    iOutVelocity    : INT;   // Speed an Antrieb (6042h) vl target velocity! EDIT the OUTPUT Setting in Expertmode
    xReady          : BOOL;  // Betriebsbereit (Operation Enabled)
    xActive         : BOOL;  // Antrieb läuft
    xError          : BOOL;  // Fehler aktiv (Bit 3)
END_VAR

VAR
    iState          : INT := 0;
END_VAR

(* Inverter settings P00.01 = 2, P00.02 = 3, P00.06 = 14, P24.00 = 1,*)

(* --- COMMAND BITS (6040h) ---         --- STATUS BITS (6041h) ---
   0: Switch on                         0: Ready to switch on
   1: Enable voltage                    1: Switched on
   2: Quick stop                        2: Operation enabled
   3: Enable operation                  3: Fault
   7: Fault reset (Flanke)              4: Voltage enabled
   8: Halt / Suspend                    5: Quick stop (0 = Active)
                                        6: Switch on disabled
*)

// 1. Modus festlegen
iModeOfOp := 2; 

// 2. Fehlerprüfung
xError := (uiStatusWord AND 16#0008) = 16#0008;

// 3. CiA 402 Zustandsmaschine
IF NOT xEnable THEN
    uiControlWord := 16#0006; // Shutdown
    iState := 0;
    xReady := FALSE;
    xActive := FALSE;
ELSIF xReset THEN
    uiControlWord := 16#0080; // Fault Reset Flanke
    iState := 0;
ELSE
    CASE iState OF
        0: // Ready to switch on?
            uiControlWord := 16#0006;
            IF (uiStatusWord AND 16#0001) = 16#0001 THEN iState := 10; END_IF
            
        10: // Switched on?
            uiControlWord := 16#0007;
            IF (uiStatusWord AND 16#0002) = 16#0002 THEN iState := 20; END_IF
            
        20: // Operation enabled?
            uiControlWord := 16#000F;
            IF (uiStatusWord AND 16#0004) = 16#0004 THEN
                xReady := TRUE;
                IF xRun THEN iState := 30; END_IF
            END_IF
            
        30: // Run (Inkl. herstellerspezifischer VFD-Bits 4,5,6 -> 0x7F)
            uiControlWord := 16#007F;
            xActive := (uiStatusWord AND 16#0004) = 16#0004;
            IF NOT xRun THEN 
                xActive := FALSE;
                iState := 20; 
            END_IF
    END_CASE
END_IF

// 4. Halt-Logik & Ausgang
IF xHalt THEN
    uiControlWord := uiControlWord OR 16#0100; // Bit 8 setzen
    iOutVelocity := 0;
ELSE
    iOutVelocity := iTargetSpeed;
END_IF
