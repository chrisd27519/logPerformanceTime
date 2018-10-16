# logPerformanceTime
Identifying and understanding potential performance issues with vRO code has always been a difficult endeavor. In most cases, debug code is added to suspect code segments to determine where potential bottlenecks exist and perhaps why the bottleneck is occurring. This type of debug code is reactionary, limited in scope and effect, and typically removed or forgotten once the immediate bottleneck is defined. What is needed is a standardized method of adding code performance measurements inline for any vRO code segment (action, workflow, scriptable task, external call, etc.) that can be consolidated and presented in a normalized way.

2.	Solution Design

The basic time measurement in vRO (and any other code engine) is done by calculating the difference between two time variables, typically startTime and endTime. Once the time measurement is available, it can be as simple as printing the value to a log to make it available.

The issues faced within vRO code with regard to time measurements are:

•	How to save the start time of a measurement when traversing multiple vRO code elements

•	How to have multiple time measurements in flight

•	How to view all time measurements and consolidate equivalent measurements graphical data.

The solution has fixed on a single vRO action to “mark” any start time and provide the time measurement when called a second time. To make multiple in flight measurements possible each time variable is identified either by the current workflow token id or by a UUID provided in the vRO action input. When the time measurement is complete a standard log entry is made that can be parsed by vRealize Log Insight to provide consolidated data views.


2.1	Standard Time Measurement Action

A vRO action named “logPerformanceTime” in the module com.vmware.pso.util has been created to provide code execution time measurements. The basic functions of this action are:

•	This action writes a log message (for log insight or other logging) that details the start time, end time and duration of a workflow.

•	The start time of the primary workflow is given by the executing workflowToken object.

•	The start time of any embedded child workflows in the primary workflow is set by calling this action with an input of "mark" (case insensitive). This marks the current position as time zero for any calls not in the primary workflow.

•	All calls of this action in child workflows will reset the time zero mark. The action must be called again to mark a time zero.

•	This action can also be used in actions used for vRA UI form inputs. The msg input format is as follows:

o	To "mark" a time zero -- mark::<msg>::UUID

o	To log the time -- end::msg::UUID

o	The keyword mark is required. Any other keyword can be used to log the time. The double colon demarcation is required. 

o	The UUID can be created in the measured action using System.nextUUID(). Be sure to send the same UUID when marking and loging time of an act

Individual start times are saved in a configuration element in the PSO /Performance folder, in the configuration element workflowTokens. Workflow tokens and UUIDs are automatically removed from this configuration element when the time measurement completes, and when the “clean” message option is used.

The logPerformanceTime action writes its output to the System.log, which places messages in both the vRO interactive-scripting log and the Server log. When parsing the messages on Log Insight, ensure only one log is used by using the filter function to specify the log, if necessary.

