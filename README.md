function generateDto(jsonObject: any, className: string, generatedClasses: Set<string> = new Set()) {
  let dtoContent = `import { ApiProperty } from '@nestjs/swagger';\n\n`;
  dtoContent += `export class ${className} {\n`;

  for (const key in jsonObject) {
    if (Array.isArray(jsonObject[key])) {
      const arrayType = typeof jsonObject[key][0] === 'object' ? capitalizeFirstLetter(key) + 'Dto' : typeof jsonObject[key][0];
      dtoContent += `  @ApiProperty({ type: [${capitalizeFirstLetter(arrayType)}] })\n`;
      dtoContent += `  ${key}: ${arrayType}[];\n\n`;

      if (typeof jsonObject[key][0] === 'object' && !generatedClasses.has(arrayType)) {
        generatedClasses.add(arrayType);
        generateDto(jsonObject[key][0], capitalizeFirstLetter(key) + 'Dto', generatedClasses);
      }
    } else if (typeof jsonObject[key] === 'object') {
      const objectType = capitalizeFirstLetter(key) + 'Dto';
      dtoContent += `  @ApiProperty({ type: ${objectType} })\n`;
      dtoContent += `  ${key}: ${objectType};\n\n`;

      if (!generatedClasses.has(objectType)) {
        generatedClasses.add(objectType);
        generateDto(jsonObject[key], objectType, generatedClasses);
      }
    } else {
      dtoContent += `  @ApiProperty()\n`;
      dtoContent += `  ${key}: ${typeof jsonObject[key]};\n\n`;
    }
  }

  dtoContent += `}\n`;

  if (!generatedClasses.has(className)) {
    console.log(dtoContent); // Exibe a DTO no console
    generatedClasses.add(className); // Marca essa classe como já gerada
  }
}

function capitalizeFirstLetter(string: string) {
  return string.charAt(0).toUpperCase() + string.slice(1);
}

// Exemplo de JSON de entrada com múltiplos objetos `month`
const jsonObject = {
  "month1": {
    "categories": ["pasta1", "pasta2"],
    "series": [
      {
        "name": "teste",
        "type": "column",
        "data": [12, 15, 16]
      }
    ]
  },
  "month2": {
    "categories": ["pasta1", "pasta2", "pasta3"],
    "series": [
      {
        "name": "teste",
        "type": "column",
        "data": [20, 22, 24]
      }
    ]
  }
};

// Gerar DTOs e exibir no console
generateDto(jsonObject, 'PrincipalDtoResponseDto');
