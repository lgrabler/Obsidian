Bsp.: 
Der Ablauf des `CellString`-Komponenten in React ist wie folgt:

1. **Komponentenparameter**: Die Komponente akzeptiert verschiedene Parameter, die durch die `CellProps`-Schnittstelle und zusätzliche `iconAttributes` spezifiziert werden. Zu diesen Parametern gehören unter anderem `options`, `column`, `value`, `rendered`, `rowIndex`, `columnIndex`, `attributes`, `allowEdit`, `onValueChange`, `eventListeners`, `cellRefs` und `iconAttributes`.
    
2. **handleChange Funktion**:
    - Diese Funktion wird aufgerufen, wenn sich der Wert des Eingabefelds (`input`) ändert.
    - Die Funktion erhält das Änderungsereignis (`ChangeEvent<HTMLInputElement>`).
    - Innerhalb der Funktion wird der neue Wert des Eingabefelds (`e.target.value`) mit der `evaluateExpression`-Funktion ausgewertet.
    - Der ausgewertete Wert wird dann durch den `onValueChange`-Callback weitergegeben.
    
3. handleFocus Funktion:
    - Diese Funktion wird aufgerufen, wenn das Eingabefeld den Fokus erhält (`FocusEvent<HTMLInputElement>`).
    - Die Funktion prüft, ob die Auswahl beim Fokus aktiviert ist, indem sie `getSelectOnFocus` mit `options` und `column` aufruft.
    - Wenn die Auswahl aktiviert ist, wird der gesamte Text im Eingabefeld ausgewählt (`e.target.select()`).
    - Abschließend wird der `onFocus`-Eventlistener aus `eventListeners` aufgerufen.
    
4. Renderlogik:
    - Die Komponente prüft, ob das Bearbeiten (`allowEdit`) aktiviert ist.
    - **Bearbeitungsmodus** (`allowEdit` ist `true`):
        - Ein `input`-Element wird zurückgegeben.
        - Das `ref`-Attribut wird verwendet, um das Eingabeelement im `cellRefs`-Objekt zu speichern.
        - Die Eventlistener und Attribute werden auf das Eingabeelement angewendet.
        - Der `value`-Prop wird auf den aktuellen Wert (`value`) gesetzt.
        - Die `handleChange`-Funktion wird dem `onChange`-Event und die `handleFocus`-Funktion dem `onFocus`-Event zugewiesen.
    - **Anzeigemodus** (`allowEdit` ist `false`):
        - Die Komponente prüft, ob `iconAttributes` vorhanden sind.
        - Wenn `iconAttributes` vorhanden sind, wird ein `span`-Element mit diesen Attributen zurückgegeben, das entweder den gerenderten Wert (`rendered`) oder den ursprünglichen Wert (`value`) anzeigt.
        - Wenn `iconAttributes` nicht vorhanden sind, wird ein `span`-Element mit den allgemeinen Attributen zurückgegeben, das entweder den gerenderten Wert (`rendered`) oder den ursprünglichen Wert (`value`) anzeigt.
        
	Diese Komponente bietet somit eine flexible Möglichkeit, Zellen entweder im Bearbeitungsmodus oder im Anzeigemodus darzustellen und unterstützt das automatische Auswählen des Textes beim Fokus sowie die Evaluierung von Ausdrücken bei Wertänderungen.
	
5. Code:
	import { evaluateExpression } from "../../utils/expressions";
	import { ChangeEvent, FocusEvent } from "react";
	import { CellProps, CellValue } from "./CellRenderer";
	import { getSelectOnFocus } from "../MultiTableContext";
	
	export default function CellString<ValueType extends CellValue>({
	    options,
	    column,
	    value,
	    rendered,
	    rowIndex,
	    columnIndex,
	    attributes,
	    allowEdit,
	    onValueChange,
	    eventListeners,
	    cellRefs,
	    iconAttributes,
	}: Readonly<CellProps<ValueType> & { iconAttributes?: { [key: string]: any } }>) {
	    const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
	        const newValue = evaluateExpression(e.target.value);
	        onValueChange(newValue);
	    };
	
	    const handleFocus = (e: FocusEvent<HTMLInputElement>) => {
	        if (getSelectOnFocus(options, column)) {
	            e.target.select();
	        }
	        eventListeners.onFocus();
	    };
	
	    if (allowEdit) {
	        return (
	            <input
	                ref={(el) => {
	                    if (!el) {
	                        return;
	                    }
	                    if (!cellRefs.current[rowIndex]) {
	                        cellRefs.current[rowIndex] = {};
	                    }
	                    cellRefs.current[rowIndex][columnIndex] = el;
	                }}
	                {...eventListeners}
	                {...attributes}
	                value={value}
	                onChange={handleChange}
	                type="text"
	                onFocus={handleFocus}
	            />
	        );
	    }
	
	    return iconAttributes ? (
	        <span {...iconAttributes}>{rendered || value}</span>
	    ) : (
	        <span {...attributes}>{rendered || value}</span>
	    );
	}