Safety Requirements:

- Emniyet gereksinimleri, FHA (Functional Hazard Assesment) / SSA (System Safety Assesment) sonucu belirlenen ve hava aracının güvenliğini etkileyen fonksiyonları doğru ve güvenilir şekilde yerine getirmesini garanti eden gereksinimlerdir.

- ARP4754A ve ARP4761 süreçleri ile bağlantılı.
  - ARP4754A: Guidelines for Development of Civil Aircraft and Systems
  - ARP4761: GUIDELINES AND METHODS FOR CONDUCTING THE SAFETY ASSESSMENT PROCESS ON CIVIL AIRBORNE SYSTEMS AND EQUIPMENT

- Genellikle emniyeti etkileyen fonksiyonel davranışı tarifler. Safety gereksinimleri constraint'de olabilir.

- Örneğin "The system shall display engine faults on the screen within 1 second."

> FHA sonucu: “Yanlış hız bilgisi göstergede pilotu yanıltabilir → Hazard = Major”
> Sistem safety requirement: “Airspeed information shall be cross-checked between ADC1 and ADC2.”
> Yazılım HLR: “The software shall compare ADC1 and ADC2 airspeed values and raise an alert if the difference exceeds 5 knots for more than 1 second.”

Eğer gereksinim safety analysis tarafından türetilmiş ve “uçuş emniyeti” için zorunlu ise → Safety requirement
Eğer gereksinim yazılımın beklenmedik durumda deterministik davranması için yazılmışsa → Robustness requirement

/************************************************************************************************************************/

Robustness Requirements:
- Robustness gereksinimleri yazılımın beklenmeyen, olağan dışı veya geçersiz girdiler ve çevresel şartlar altında emniyetli bir şekilde davranmasını garanti altına alan gereksinimlerdir.

- Sistemi bozabilecek anormal yada hatalı girişlere karşı yazılımın kontrollü bir şekilde tepki vermesini ele alır.

- Input validation, exception handling, boundary condition management

- Örneğin: "WHEN the temperatureSensor value is out of range, The Software shall use the last valid valie and set sensorValueInvalidFlag"


Robustness Requirements Checklist:
1. Input Data Validation
   - Have you defined the valid range for each input?
   - Is there a requirement for handling out-of-range values?
   - Is there a requirement for handling missing or stale data?
   - Is there a requirement for handling invalid/corrupted messages (e.g. checksum/CRC errors)?
   - Is there a requirement for handling unexpected data size or format?
   
2. Boundary Conditions
   - Are there explicit requirements for minimum and maximun input values?
   - Is there a requirement for clamping/saturating inputs at boundaries?
   - Is negative or zero input explicity handled (when not allowed)?
   
3. Computation Robustness
   - Are there requirements for handling division by zero?
   - Are there requirements for handling overflow/underflow in arithmetic operations?
   - Are there requirements for handling NaN or infinity results from computations?
   - Is there a requirements for handling invalid intermediate states (e.g., uninitialized variables)?
   
4. Timing Robustness
   - What happens if input data arrives too late (timeout)?
   - What happens if input data arrives too early or too fast (rate checks)?
   - Is there a requirement for handling task overruns (execution time exceeding budget)?

5. Resource Handling
   - Is there a requirement for handling buffer overflow/underflow?
   - Is there a requirement for handling memory allocation failures?
   - Is there a requirement for handling stack overflow or excessive recursion (if applicable)?
   
6. Communication Robustness
   - Is there a requirement for handling unexpected sender IDs or source addresses?
   - Is there a requirement for handling invalid sequenve counters in messages?
   - Is there a requirement for handling duplicate or out-of-order messages?
   
7. Fail-Safe / Fallback Behavior
   - If data is invalid, does the system default to last known good value?
   - If data is missing, does the system set a safe default?
   - Is there a requirement for the software to signal an error flag or status when abnormal conditions occur?
   - Is there a requirement for the system to transition to a safer state under unrecoverable errors?
   
8. General Verifiability
   - Is every robustness requirement specific, measurable, and testable?
   - Is the behavior in abnormal conditions deterministic and predictable?
   - Are robustness requirements traceable to safety objectives or software quality objectives?
   
   

Examples:

Input Range Handling:
   WHEN the followings are satisfied: (Yada burada eğer DD'te tange tanımlıyasa: "WHEN the pressureInput is out-of-range")
   - the pressureInput < TBD, OR
   - the pressureInput > TBD
   The Software perform followings:
   - reject the pressureInput value
   - set pressureInputStatus to INVALID
   
Default or Safe Value:
   WHEN the sensorReadStatus = INVALID,
   The Software shall NOT set temperatureSensorInput.
   
Timeout:
	WHILE task = sendTask,
	WHEN no acknowledgeMessage is received within 0.5 second,
	The Software proform followings:
	- set commFault to True
	- set task to commErrorHandleTask
	
Error Detection and Reporting:
	WHEN messageCRCERRORCount = 3
	The Software shall perform followings:
	- set communicationStatus to FAILURE
	- send communicationStatus to the display
	
Graceful Degredation:
   WHEN the followings are satisfied:
   - dataInputStatus is INVALID, AND
   - redundantInpuStatus is VALID
   The Software shall perform followings:
   - set ADCDataInput to redundantADCDataInput
   - set ADCDataStatus to DEGRADED
   
Boundary Conditions:
   WHEN the temperatureValue exceed TEMP_MAX,
   The software shall saturate temperatureValue to TEMP_MAX.
   
Unexpected Sequences:
	WHILE updating inportDataMessafe,
	WHEN two consecutive inportDataMessafe message contain identical timetsamp field, The Software shall ignore second inportDataMessafe message.
	
Resource Exhaustion:
	WHEN the execution of the sendTask exceed sendTaskTimeBudged by 50ns,
	The Software shall perform followings in given order:
	- terminate the sendTask,
	- set sendTaskTimeout to TURE,
	- continue with the nextTask
	
Recovery:
   WHEN commandMessage = resetFaults
   The Software perform followings:
   - set communicationStatus to NO_ERROR
   - set task to startTask
   
