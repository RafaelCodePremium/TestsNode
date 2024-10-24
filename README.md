generateDto(jsonObject: any, className: string) {
  const generatedClasses = new Set<string>(); // Set para rastrear as classes já geradas

  function processDto(object: any, name: string) {
    let dtoContent = `import { ApiProperty } from '@nestjs/swagger';\n\n`;
    dtoContent += `export class ${name} {\n`;

    for (const key in object) {
      if (Array.isArray(object[key])) {
        const arrayType = typeof object[key][0] === 'object' ? capitalizeFirstLetter(key) + 'Dto' : typeof object[key][0];
        dtoContent += `  @ApiProperty({ type: [${capitalizeFirstLetter(arrayType)}] })\n`;
        dtoContent += `  ${key}: ${arrayType}[];\n\n`;

        if (typeof object[key][0] === 'object' && !generatedClasses.has(arrayType)) {
          generatedClasses.add(arrayType);
          processDto(object[key][0], capitalizeFirstLetter(key) + 'Dto');
        }
      } else if (typeof object[key] === 'object') {
        const objectType = capitalizeFirstLetter(key) + 'Dto';
        dtoContent += `  @ApiProperty({ type: ${objectType} })\n`;
        dtoContent += `  ${key}: ${objectType};\n\n`;

        if (!generatedClasses.has(objectType)) {
          generatedClasses.add(objectType);
          processDto(object[key], objectType);
        }
      } else {
        dtoContent += `  @ApiProperty()\n`;
        dtoContent += `  ${key}: ${typeof object[key]};\n\n`;
      }
    }

    dtoContent += `}\n`;

    if (!generatedClasses.has(name)) {
      console.log(dtoContent); // Exibe a DTO no console
      generatedClasses.add(name); // Marca essa classe como já gerada
    }
  }

  processDto(jsonObject, className); // Inicia o processo de geração da DTO
}

function capitalizeFirstLetter(string: string) {
  return string.charAt(0).toUpperCase() + string.slice(1);
}
